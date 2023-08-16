## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Removing redundant require() can save gas](https://github.com/code-423n4/2021-09-swivel-findings/issues/111) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The require(matureMarket(u, m) is redundant because matureMarket always returns true and reverts if any of its require() check fails.

Removing this can save a little gas.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L127

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/marketplace/MarketPlace.sol#L75-L91

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Remove redundant require()

