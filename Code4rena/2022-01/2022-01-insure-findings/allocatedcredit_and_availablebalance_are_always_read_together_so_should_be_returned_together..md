## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [allocatedCredit and availableBalance are always read together so should be returned together.](https://github.com/code-423n4/2022-01-insure-findings/issues/37) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

It seems that we always want to get a pool's `allocatedCredit` and `availableBalance` together, suggesting that these values are tightly coupled. 

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L284-L287

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L356-L360

If we're regularly going to be requesting these values together it may be worth considering having a single function in the pool template which returns both of these values. This would save gas costs of performing an extra external call to the pool contract.

## Recommended Mitigation Steps

Consider having a function which returns both of these values to avoid repeated calls into the same contract for related info.

