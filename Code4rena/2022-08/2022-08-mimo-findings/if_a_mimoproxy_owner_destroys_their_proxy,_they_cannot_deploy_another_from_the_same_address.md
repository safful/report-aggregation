## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [If a MIMOProxy owner destroys their proxy, they cannot deploy another from the same address](https://github.com/code-423n4/2022-08-mimo-findings/issues/162) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyRegistry.sol#L45-L59


# Vulnerability details

When deploying a new `MIMOProxy`, the `MIMOProxyRegistry` first checks whether a proxy exists with the same owner for the given address. If an existing proxy is found, the deployment reverts:

[`MIMOProxyRegistry#deployFor`](https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyRegistry.sol#L45-L59)

```solidity
  function deployFor(address owner) public override returns (IMIMOProxy proxy) {
    IMIMOProxy currentProxy = _currentProxies[owner];

    // Do not deploy if the proxy already exists and the owner is the same.
    if (address(currentProxy) != address(0) && currentProxy.owner() == owner) {
      revert CustomErrors.PROXY_ALREADY_EXISTS(owner);
    }

    // Deploy the proxy via the factory.
    proxy = factory.deployFor(owner);

    // Set or override the current proxy for the owner.
    _currentProxies[owner] = IMIMOProxy(proxy);
  }
}
```

However, if a `MIMOProxy` owner intentionally or accidentally destroys their proxy by `delegatecall`ing a target that calls `selfdestruct`, the address of their destroyed proxy will remain in the `_currentProxies` mapping, but the static call to `currentProxy.owner()` on L49 will revert. The caller will be blocked from deploying a new proxy from the same address that created their original `MIMOProxy.

**Impact:** If a user accidentally destroys their MIMOProxy, they must use a new EOA address to deploy another.

### Recommendation

Check whether the proxy has been destroyed as part of the "proxy already exists" conditions. If the proxy address has a codesize of zero, it has been destroyed:

```solidity
    // Do not deploy if the proxy already exists and the owner is the same.
    if (address(currentProxy) != address(0) && currentProxy.code.length > 0 && currentProxy.owner() == owner) {
      revert CustomErrors.PROXY_ALREADY_EXISTS(owner);
    }

```

### Test cases

We'll use this `ProxyAttacks` helper contract to manipulate proxy storage. Note that it has the same storage layout as `MIMOProxy`.

```solidity
contract ProxyAttacks {

   address public owner;
   uint256 public minGasReserve;
   mapping(address => mapping(address => mapping(bytes4 => bool))) internal _permissions;

   // Selector 0x9cb8a26a
   function selfDestruct() external {
     selfdestruct(payable(address(0)));
   }
}
```

Then deploy the `ProxyAttacks` helper in a test environment and use `MIMOProxy` to `delegatecall` into it:

```typescript
import chai, { expect } from 'chai';
import { solidity } from 'ethereum-waffle';
import { deployments, ethers } from 'hardhat';

import { MIMOProxy, MIMOProxyFactory, MIMOProxyRegistry, ProxyAttacks } from '../../typechain';

chai.use(solidity);

const setup = deployments.createFixture(async () => {
  const { deploy } = deployments;
  const [owner, attacker] = await ethers.getSigners();

  await deploy("MIMOProxy", {
    from: owner.address,
    args: [],
  });
  const mimoProxyBase: MIMOProxy = await ethers.getContract("MIMOProxy");

  await deploy("MIMOProxyFactory", {
    from: owner.address,
    args: [mimoProxyBase.address],
  });
  const mimoProxyFactory: MIMOProxyFactory = await ethers.getContract("MIMOProxyFactory");

  await deploy("MIMOProxyRegistry", {
    from: owner.address,
    args: [mimoProxyFactory.address],
  });
  const mimoProxyRegistry: MIMOProxyRegistry = await ethers.getContract("MIMOProxyRegistry");

  await deploy("ProxyAttacks", {
    from: owner.address,
    args: [],
  });
  const proxyAttacks: ProxyAttacks = await ethers.getContract("ProxyAttacks");

  return {
    owner,
    attacker,
    mimoProxyBase,
    mimoProxyFactory,
    mimoProxyRegistry,
    proxyAttacks,
  };
});

describe("Proxy attack tests", () => {
  it("Proxy instance self destruct + recreation", async () => {
    const { owner, mimoProxyRegistry, proxyAttacks } = await setup();
    await mimoProxyRegistry.deploy();
    const currentProxy = await mimoProxyRegistry.getCurrentProxy(owner.address);
    const proxy = await ethers.getContractAt("MIMOProxy", currentProxy);

    // Delegatecall to selfDestruct on ProxyAttacks contract
    await proxy.execute(proxyAttacks.address, "0x9cb8a26a");

    // Owner's existing proxy is destroyed
    expect(proxy.owner()).to.be.revertedWith("call revert exception");

    // Cannot deploy another proxy for this address through the registry
    await expect(mimoProxyRegistry.deploy()).to.be.revertedWith("function returned an unexpected amount of data");
  });
});
```