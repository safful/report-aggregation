## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Router Owner Could Be Rugged By Admin](https://github.com/code-423n4/2022-06-connext-findings/issues/150) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L293
https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L490
https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L212


# Vulnerability details

## Proof-of-Concept

Assume that Alice's router has large amount of liquidity inside.

Assume that the Connext Admin decided to remove a router owned by Alice. The Connext Admin will call the `RoutersFacet.removeRouter` function, and all information related to Alice's router will be erased (set to 0x0) from the `s.routerPermissionInfo`.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L293](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L293)

```solidity
  function removeRouter(address router) external onlyOwner {
    // Sanity check: not empty
    if (router == address(0)) revert RoutersFacet__removeRouter_routerEmpty();

    // Sanity check: needs removal
    if (!s.routerPermissionInfo.approvedRouters[router]) revert RoutersFacet__removeRouter_notAdded();

    // Update mapping
    s.routerPermissionInfo.approvedRouters[router] = false;

    // Emit event
    emit RouterRemoved(router, msg.sender);

    // Remove router owner
    address _owner = s.routerPermissionInfo.routerOwners[router];
    if (_owner != address(0)) {
      emit RouterOwnerAccepted(router, _owner, address(0));
      // delete routerOwners[router];
      s.routerPermissionInfo.routerOwners[router] = address(0);
    }

    // Remove router recipient
    address _recipient = s.routerPermissionInfo.routerRecipients[router];
    if (_recipient != address(0)) {
      emit RouterRecipientSet(router, _recipient, address(0));
      // delete routerRecipients[router];
      s.routerPermissionInfo.routerRecipients[router] = address(0);
    }

    // Clear any proposed ownership changes
    s.routerPermissionInfo.proposedRouterOwners[router] = address(0);
    s.routerPermissionInfo.proposedRouterTimestamp[router] = 0;
  }
```

Alice is aware that her router has been removed by Connext Admin, so she decided to withdraw the liquidity from her previous router by calling `RoutersFacet.removeRouterLiquidityFor`.

However, when Alice called the `RoutersFacet.removeRouterLiquidityFor` function, it will revert every single time. This is because the condition `msg.sender != getRouterOwner(_router)` will always fail.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L490](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L490)

```solidity
  /**
   * @notice This is used by any router owner to decrease their available liquidity for a given asset.
   * @param _amount - The amount of liquidity to remove for the router
   * @param _local - The address of the asset you're removing liquidity from. If removing liquidity of the
   * native asset, routers may use `address(0)` or the wrapped asset
   * @param _to The address that will receive the liquidity being removed
   * @param _router The address of the router
   */
  function removeRouterLiquidityFor(
    uint256 _amount,
    address _local,
    address payable _to,
    address _router
  ) external nonReentrant whenNotPaused {
    // Caller must be the router owner
    if (msg.sender != getRouterOwner(_router)) revert RoutersFacet__removeRouterLiquidityFor_notOwner();

    // Remove liquidity
    _removeLiquidityForRouter(_amount, _local, _to, _router);
  }
```

Since the `RoutersFacet.removeRouter` function has earlier erased all information related to Alice's router within `s.routerPermissionInfo`, the `getRouterOwner` function will always return the router address.

In this case, the router address will not match against `msg.sender` address/Alice address, thus Alice attempts to call `removeRouterLiquidityFor` will always revert.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L212](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/RoutersFacet.sol#L212)

```solidity
  function getRouterOwner(address _router) public view returns (address) {
    address _owner = s.routerPermissionInfo.routerOwners[_router];
    return _owner == address(0) ? _router : _owner;
  }
```

## Impact

Router owner who provides liquidity could be rugged by Connext admin. When this happen, the router owner funds will be struck  within the `RoutersFacet` contract, and there is no way for the router owner to retrieve their liquidity.

In the worst case scenario, a compromised Connext admin could remove all routers, and cause all liquidity to be struck within `RoutersFacet` and no router owner could withdraw their liquidity from the contract. Next, the `RouterFacet` contract could be upgraded to include additional function to withdraw all liquidity from the contract to an arbitrary wallet address.

## Recommended Mitigation Steps

The router owner is still entitled to their own liquidity even though their router has been removed by Connext Admin. Thus, they should be given the right to take back their liquidity when such an event happens. The contract should update its implementation to support this. This will give more assurance to the router owner.

