## Tags

- bug
- 1 (Low Risk)
- mStableYieldSource
- SwappableYieldSource
- sponsor confirmed

# [Missing zero-address checks](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/41) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Zero-address checks as input validation closest to the function beginning is a best-practice. There are two places where an explicit zero-address check is missing which may lead to a later revert, gas wastage or even token burn.

## Proof of Concept

1. Explicit zero-address check is missing here for _newYieldSource
 and will revert later down the control flow on L256: https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L269

https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L256


2. Missing zero-address check on ‘to’ address will lead to token burn because imBalances accounts it for the zero-address from which it can never be redeemed using msg.sender: https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L85

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L94


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add explicit zero-address checks closest to the function entry.

