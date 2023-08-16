## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)
- resolved

# [Param `initInterestRate` in `TempusPool::constructor` should not be 0](https://github.com/code-423n4/2021-10-tempus-findings/issues/12) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
If `initInterestRate` in the `TempusPool`'s constructor is given as 0, 
no funds can be withdrawn as `getRedemptionAmounts()` always panic errors with
_division by 0_ ([link](https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/TempusPool.sol#L305)).

## Recommended Mitigation Steps
It should be stated in the constructor's specs that `initInterestRate` should
not be 0.

