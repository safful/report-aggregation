## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Unnecessary `balanceOfWant() > 0`](https://github.com/code-423n4/2021-09-yaxis-findings/issues/141) 

# Handle

0xsanson


# Vulnerability details

## Impact
During the `_harvest` function in NativeStrategyCurve3Crv.sol, there's a call to `_deposit()` only `if (balanceOfWant() > 0)`. This if-statement can be removed since `_deposit` calculates again `balanceOfWant()` and makes the same check.
This way the function saves a `.balanceOf` call.

## Proof of Concept
https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/strategies/NativeStrategyCurve3Crv.sol#L123

## Tools Used
editor

