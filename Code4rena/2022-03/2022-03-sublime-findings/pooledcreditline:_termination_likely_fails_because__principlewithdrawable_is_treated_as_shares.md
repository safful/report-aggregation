## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [PooledCreditLine: termination likely fails because _principleWithdrawable is treated as shares](https://github.com/code-423n4/2022-03-sublime-findings/issues/21) 

# Lines of code

https://github.com/sublime-finance/sublime-v1/blob/46536a6d25df4264c1b217bd3232af30355dcb95/contracts/PooledCreditLine/LenderPool.sol#L404-L406


# Vulnerability details

## Details & Impact

`_principalWithdrawable` is denominated in the borrowAsset, but subsequently treats it as the share amount to be withdrawn.

```jsx
// _notBorrowed = borrowAsset amount that isn't borrowed
// totalSupply[_id] = ERC1155 total supply of _id
// _borrowedTokens = borrower's specified borrowLimit
uint256 _principalWithdrawable = _notBorrowed.mul(totalSupply[_id]).div(_borrowedTokens);

SAVINGS_ACCOUNT.withdrawShares(_borrowAsset, _strategy, _to, _principalWithdrawable.add(_totalInterestInShares), false);
```

## Recommended Mitigation Steps

The amount of shares to withdraw can simply be `_sharesHeld`.

Note that this comes with the assumption that `terminate()` is only called when the credit line is `ACTIVE` or `EXPIRED` (consider ensuring this condition on-chain), because `_sharesHeld` **excludes principal withdrawals,** so the function will fail once a lender withdraws his principal.

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

