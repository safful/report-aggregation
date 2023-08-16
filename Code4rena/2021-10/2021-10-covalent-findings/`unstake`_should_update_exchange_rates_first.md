## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`unstake` should update exchange rates first](https://github.com/code-423n4/2021-10-covalent-findings/issues/57) 

# Handle

cmichel


# Vulnerability details

The `unstake` function does not immediately update the exchange rates. It first computes the `validatorSharesRemove = tokensToShares(amount, v.exchangeRate)` **with the old exchange rate**.

Only afterwards, it updates the exchange rates (if the validator is not disabled):

```solidity
// @audit shares are computed here with old rate
uint128 validatorSharesRemove = tokensToShares(amount, v.exchangeRate);
require(validatorSharesRemove > 0, "Unstake amount is too small");

if (v.disabledEpoch == 0) {
    // @audit rates are updated here
    updateGlobalExchangeRate();
    updateValidator(v);
    // ...
}
```

## Impact
More shares for the amount are burned than required and users will lose rewards in the end.

## POC
Demonstrating that users will lose rewards:

1. Assume someone staked `1000 amount` and received `1000 shares`, and `v.exchangeRate = 1.0`. (This user is the single staker)
2. Several epochs pass, interest accrues, and `1000 tokens` accrue for the validator, `tokensGivenToValidator = 1000`. User should be entitled to 1000 in principal + 1000 in rewards = 2000 tokens.
3. But user calls `unstake(1000)`, which sets `validatorSharesRemove = tokensToShares(amount, v.exchangeRate) = 1000 / 1.0 = 1000`. **Afterwards**, the exchange rate is updated: `v.exchangeRate += tokensGivenToValidator / totalShares = 1.0 + 1.0 = 2.0`. The staker is updated with `s.shares -= validatorSharesRemove = 0` and `s.staked -= amount = 0`. And the user receives their 1000 tokens but notice how the user's shares are now at zero as well.
4. User tries to claim rewards calling `redeemAllRewards` which fails as the `rewards` are 0.

If the user had first called `redeemAllRewards` and `unstake` afterwards they'd have received their 2000 tokens.

## Recommended Mitigation Steps
The exchange rates always need to be updated first before doing anything.
Move the `updateGlobalExchangeRate()` and `updateValidator(v)` calls to the beginning of the function.


