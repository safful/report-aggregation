## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- YearnV2YieldSource

# [Zero-address check unnecessary due to the initializer modifier](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/42) 

# Handle

0xRajeev


# Vulnerability details

## Impact

YearnV2YieldSource initialize does a zero-address check for value address to detect if it has already been initialized. This is an unnecessary check because vault address default value is zero, it is not initialized/set anywhere else and the initializer modifier will prevent the calling of initialize() a second time. So vault is guaranteed to be zero in initialize().

The impact is gas wastage from an additional SLOAD of vault state variable and the require() check.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L25

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L73

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove the zero-address check for vault.

