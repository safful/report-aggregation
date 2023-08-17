## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Repaying AAVE Loan in `_local` rather than `adopted` asset](https://github.com/code-423n4/2022-06-connext-findings/issues/103) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L80



# Vulnerability details

## Impact

When repaying the AAVE Portal in [`repayAavePortal()`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L80) the `_local` asset is used to repay the loan in `_backLoan()` rather than the `adopted` asset. This is likely to cause issues in production when actually repaying loans if the asset/token being repayed to AAVE is not the same as the asset/token that was borrowed.

## Proof of Concept
The comment on [`L93`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L93) of [`PortalFacet.sol`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol) states;

```
// Need to swap into adopted asset or asset that was backing the loan
// The router will always be holding collateral in the local asset while the loaned asset
// is the adopted asset
```

The swap is executed on [`L98`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L98) in the call to `AssetLogic.swapFromLocalAssetIfNeededForExactOut()` however the return value `adopted` is never used (it's an unused local variable). The full function is shown below;

```
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
```
The balance of the `_local` token is reduced but instead of the `adopted` token being passed to [`_backLoan()`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L112) in L112 the `_local` token is used.
## Tools Used
Vim

## Recommended Mitigation Steps
To be consistent with the comments in the [`repayAavePortal()`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L80) function `adopted` should be passed to `_backLoan` so that the loan is repayed in the appropriate token.

Remove the reference to `_local` in the [`_backLoan()`](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/PortalFacet.sol#L112) function and replace it with `adopted` so it reads;

`_backLoan(adopted, _backingAmount, _feeAmount, _transferId);`

