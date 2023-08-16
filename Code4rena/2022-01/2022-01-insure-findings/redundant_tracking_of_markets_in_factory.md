## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Redundant tracking of markets in factory](https://github.com/code-423n4/2022-01-insure-findings/issues/39) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

Here we push a new market onto an array in the factory whilst we just added the market to the registry.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L214-L216

## Recommended Mitigation Steps

This array on the factory seems redundant and so it can be removed.

