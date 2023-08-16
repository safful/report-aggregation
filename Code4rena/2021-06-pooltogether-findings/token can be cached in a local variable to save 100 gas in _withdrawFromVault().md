## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [token can be cached in a local variable to save 100 gas in _withdrawFromVault()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/45) 

# Handle

0xRajeev


# Vulnerability details

## Impact

token state variable is used in two places in _withdrawFromVault(). It can be cached in a local variable  at the beginning of the function to save 100 gas from one repeated SLOAD.


## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L185

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L192


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache token in a local variable  at the beginning of the function and use that instead.

