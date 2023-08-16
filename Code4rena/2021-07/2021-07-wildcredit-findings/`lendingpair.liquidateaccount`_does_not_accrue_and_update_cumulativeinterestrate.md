## Tags

- bug
- disagree with severity
- sponsor confirmed
- 3 (High Risk)

# [`LendingPair.liquidateAccount` does not accrue and update cumulativeInterestRate](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/122) 

# Handle

cmichel


# Vulnerability details

The `LendingPair.liquidateAccount` function does not accrue and update the `cumulativeInterestRate` first, it only calls `_accrueAccountInterest` which does not update and instead uses the old `cumulativeInterestRate`.

## Impact
The liquidatee (borrower)'s state will not be up to date.
I could skip some interest payments by liquidating myself instead of repaying if I'm under-water.
As the market interest index is not accrued, the borrower does not need to pay any interest accrued from the time of the last accrual until now.

## Recommendation
It should call `accrueAccount` instead of `_accrueAccountInterest`


