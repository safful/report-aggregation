## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary market status check on redemption](https://github.com/code-423n4/2022-01-insure-findings/issues/32) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

gas costs

## Proof of Concept

Here on L563 we check the market status however we have already done this on L558

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L557-L567

## Recommended Mitigation Steps

Remove redundant check (check other market templates as well)

