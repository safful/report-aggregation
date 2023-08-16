## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Users could shift tokens on `Staker` with more than he has staked](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/141) 

# Handle

shw


# Vulnerability details

## Impact

The `shiftTokens` function of `Staker` checks whether the user has staked at least the number of tokens he wants to shift from one side to the other (line 885). A user could call the `shiftTokens` function multiple times before the next price update to shift the staker's token from one side to the other with more than he has staked.

## Proof of Concept

Referenced code:
[Staker.sol#L885](https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/Staker.sol#L885)

## Recommended Mitigation Steps

Add checks on `userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_long` and `userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_short` to ensure that the sum of the two variables does not exceed user's stake balance.

