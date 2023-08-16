## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [LenderPool: Principal withdrawable is incorrectly calculated if start() is invoked with non-zero start fee](https://github.com/code-423n4/2022-03-sublime-findings/issues/19) 

# Lines of code

https://github.com/sublime-finance/sublime-v1/blob/46536a6d25df4264c1b217bd3232af30355dcb95/contracts/PooledCreditLine/LenderPool.sol#L594-L599
https://github.com/sublime-finance/sublime-v1/blob/46536a6d25df4264c1b217bd3232af30355dcb95/contracts/PooledCreditLine/LenderPool.sol#L399-L404


# Vulnerability details

## Details & Impact

The `_principalWithdrawable` calculated will be more than expected if `_start()` is invoked with a non-zero start fee, because the borrow limit is reduced by the fee, resulting in `totalSupply[id]` not being 1:1 with the borrow limit.

```jsx
function _calculatePrincipalWithdrawable(uint256 _id, address _lender) internal view returns (uint256) {
  uint256 _borrowedTokens = pooledCLConstants[_id].borrowLimit;
  uint256 _totalLiquidityWithdrawable = _borrowedTokens.sub(POOLED_CREDIT_LINE.getPrincipal(_id));
  uint256 _principalWithdrawable = _totalLiquidityWithdrawable.mul(balanceOf(_lender, _id)).div(_borrowedTokens);
  return _principalWithdrawable;
}
```

## Proof of Concept

Assume the following conditions:

- Alice, the sole lender, provided `100_000` tokens: `totalSupply[_id] = 100_000`
- `borrowLimit = 99_000` because of a 1% startFee
- Borrower borrowed zero amount

When Alice attempts to withdraw her tokens, the `_principalWithdrawable` amount is calculated as 

```jsx
_borrowedTokens = 99_000
_totalLiquidityWithdrawable = 99_000 - 0 = 99_000
_principalWithdrawable = 99_000 * 100_000 / 99_000 = 100_000
```

This is more than the available principal amount of `99_000`, so the withdrawal will fail.

## Recommended Mitigation Steps

One hack-ish way is to save the initial supply in `minBorrowAmount` (perhaps rename the variable to `minInitialSupply`) when the credit line is accepted, and replace `totalSupply[_id]` with it. 

The other places where `minBorrowAmount` are used will not be affected by the change because:

- startTime has been zeroed, so `start()` cannot be invoked (revert with error S1)
- credit line status would have been changed to `ACTIVE` and cannot be changed back to `REQUESTED`, meaning the check below will be false regardless of the value of `minBorrowAmount`.
    
    ```jsx
    _status == PooledCreditLineStatus.REQUESTED &&
    block.timestamp > pooledCLConstants[_id].startTime &&
    totalSupply[_id] < pooledCLConstants[_id].minBorrowAmount
    ```
    

Code amendment example:

```jsx

function _accept(uint256 _id, uint256 _amount) internal {
  ...
  // replace delete pooledCLConstants[_id].minBorrowAmount; with the following:
  pooledCLConstants[_id].minInitialSupply = totalSupply[_id];
}

// update comment in _withdrawLiquidity
// Case 1: Pooled credit line never started because desired amount wasn't reached
// state will never revert back to REQUESTED if credit line is accepted so this case is never run

function _calculatePrincipalWithdrawable(uint256 _id, address _lender) internal view returns (uint256) {
  uint256 _borrowedTokens = pooledCLConstants[_id].borrowLimit;
  uint256 _totalLiquidityWithdrawable = _borrowedTokens.sub(POOLED_CREDIT_LINE.getPrincipal(_id));
  // totalSupply[id] replaced with minInitialSupply
  uint256 _principalWithdrawable = _totalLiquidityWithdrawable.mul(balanceOf(_lender, _id)).div(minInitialSupply);
  return _principalWithdrawable;
}
```

In `terminate()`, the shares withdrawable can simply be `_sharesHeld`.

```jsx
function terminate(uint256 _id, address _to) external override onlyPooledCreditLine nonReentrant {
  address _strategy = pooledCLConstants[_id].borrowAssetStrategy;
  address _borrowAsset = pooledCLConstants[_id].borrowAsset;
  uint256 _sharesHeld = pooledCLVariables[_id].sharesHeld;

  SAVINGS_ACCOUNT.withdrawShares(_borrowAsset, _strategy, _to, _sharesHeld, false);
  delete pooledCLConstants[_id];
  delete pooledCLVariables[_id];
}
```

