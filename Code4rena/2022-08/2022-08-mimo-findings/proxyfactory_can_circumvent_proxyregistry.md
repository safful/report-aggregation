## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ProxyFactory can circumvent ProxyRegistry](https://github.com/code-423n4/2022-08-mimo-findings/issues/123) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/eb1a5016b69f72bc1e4fd3600a65e908bd228f13/contracts/proxy/MIMOProxyFactory.sol#L45


# Vulnerability details

## Impact
The `deployFor()` function in `MIMOProxyFactory.sol` can be called directly instead of being called within `MIMOProxyRegistry.sol`. This results in the ability to create many MIMOProxies that are not registered within the registry. The proxies deployed directly through the factory will lack the ability to call certain actions such as leveraging and emptying the vault, but will be able to call all functions in `MIMOVaultAction.sol`.

This inconsistency doesn't feel natural and would be remedied by adding an `onlyRegistry` modifier to the `ProxyFactory.deployFor()` function.

## Proof of Concept
`MIMOProxyFactory.deployFor()` lacking any access control:
```
  function deployFor(address owner) public override returns (IMIMOProxy proxy) {
    proxy = IMIMOProxy(mimoProxyBase.clone());
    proxy.initialize();


    // Transfer the ownership from this factory contract to the specified owner.
    proxy.transferOwnership(owner);


    // Mark the proxy as deployed.
    _proxies[address(proxy)] = true;


    // Log the proxy via en event.
    emit DeployProxy(msg.sender, owner, address(proxy));
  }
}
```

Example of reduced functionality: `MIMOEmptyVault.executeOperation()` checks proxy existence in the proxy registry therefore can't be called.
```
  function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) external override returns (bool) {
    (address owner, uint256 vaultId, SwapData memory swapData) = abi.decode(params, (address, uint256, SwapData));
    IMIMOProxy mimoProxy = IMIMOProxy(proxyRegistry.getCurrentProxy(owner));
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
Adding access control to ensure that the factory deployFor function is called from the proxy registry would mitigate this issue.