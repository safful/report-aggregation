## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`YearnV2YieldSource` wrong subtraction in withdraw](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/90) 

# Handle

cmichel


# Vulnerability details

`YearnV2YieldSource._withdrawFromVault` uses a wrong subtraction.
When withdrawing from the `vault` one redeems `yTokens` for `token`s, thus the `token` balance of the contract should increase after withdrawal.
But the contract subtracts the `currentBalance` from the `previousBalance`:

```solidity
uint256 yShares = _tokenToYShares(amount);
uint256 previousBalance = token.balanceOf(address(this));
// we accept losses to avoid being locked in the Vault (if losses happened for some reason)
if(maxLosses != 0) {
    vault.withdraw(yShares, address(this), maxLosses);
} else {
    vault.withdraw(yShares);
}
uint256 currentBalance = token.balanceOf(address(this));
// @audit-issue this seems wrong
return previousBalance.sub(currentBalance);
```

## Impact

All vault withdrawals fail due to the integer underflow as the `previousBalance` is less than `currentBalance`. Users won't be able to get back their investment.

## Recommended Mitigation Steps

It should return `currentBalance > previousBalance ? currentBalance - previousBalance : 0`

