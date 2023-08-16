## Tags

- bug
- G (Gas Optimization)
- SwappableYieldSource
- sponsor confirmed

# [Changing function visibility from public to external can save gas](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/32) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Assuming the initialize() function is going to be called from a deployment script, its visibility can be made external. 

For public functions, the input parameters are copied to memory automatically which costs gas. If a function is only called externally, making its visibility as external will save gas because external function’s parameters are not copied into memory and are instead read from calldata directly.

## Proof of Concept

https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L98-L104


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change visibility to external.

