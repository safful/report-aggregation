## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- SushiYieldSource
- BadgerYieldSource

# [Declare functions as `external` to save gas](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/107) 

# Handle

shw


# Vulnerability details

## Impact

Functions (e.g., `supplyTokenTo`, `redeemToken`) in the `BadgerYieldSource` and `SushiYieldSource` can be declared `external` instead of `public` to save gas.

## Proof of Concept

Referenced code:
[BadgerYieldSource.sol#L26](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L26)
[BadgerYieldSource.sol#L32](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L32)
[BadgerYieldSource.sol#L43](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L43)
[BadgerYieldSource.sol#L57](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/BadgerYieldSource.sol#L57)
[SushiYieldSource.sol#L29](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L29)
[SushiYieldSource.sol#L35](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L35)
[SushiYieldSource.sol#L47](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L47)
[SushiYieldSource.sol#L66](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/SushiYieldSource.sol#L66)

## Recommended Mitigation Steps

Change the keyword `public` to `external`.

