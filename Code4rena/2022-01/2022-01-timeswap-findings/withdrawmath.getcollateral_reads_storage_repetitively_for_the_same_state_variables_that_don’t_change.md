## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [WithdrawMath.getCollateral reads storage repetitively for the same state variables that don’t change](https://github.com/code-423n4/2022-01-timeswap-findings/issues/95) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on state variables storage access.

## Proof of Concept

getCollateral only reads state.reserves.asset, state.totalClaims.insurance and state.reserves.collateral up to 2 times each, and state.totalClaims.bond up to 4 times:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/libraries/WithdrawMath.sol#L26


## Recommended Mitigation Steps

Save all four state variables to memory before running the logic.


