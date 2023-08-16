## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [More accurate calculation of return USD of `withdrawSingleByLiquidity`](https://github.com/code-423n4/2021-06-gro-findings/issues/121) 

# Handle

shw


# Vulnerability details

## Impact

The `withdrawSingleByLiquidity` function of `LifeGuard3Pool` calls `buoy.singleStableToUsd` to calculate the return USD amount, which internally calls `_stableToUsd` with the `deposit` parameter set to `true`. A more accurate calculation is to set the `deposit` parameter to `false` since this action is a withdrawal. A similar issue exists in the function `calcProtocolWithdraw` of `Allocation`, where the current strategy's USD is calculated by `buoy.singleStableToUsd`.

## Proof of Concept

Referenced code:
[LifeGuard3Pool.sol#L226](https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pools/LifeGuard3Pool.sol#L226)
[Buoy3Pool.sol#L122](https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pools/oracle/Buoy3Pool.sol#L122)
[Allocation.sol#L142](https://github.com/code-423n4/2021-06-gro/blob/main/contracts/insurance/Allocation.sol#L142)

## Recommended Mitigation Steps

Consider adding a new boolean parameter, `deposit`, to the `singleStableToUsd` function of `Buoy3Pool` to indicate whether the action is a deposit or not, as that in the `stableToUsd` and `stableToLp` functions.

