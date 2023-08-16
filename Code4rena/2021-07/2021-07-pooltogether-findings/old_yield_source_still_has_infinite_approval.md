## Tags

- bug
- 2 (Med Risk)
- SwappableYieldSource
- sponsor confirmed

# [Old yield source still has infinite approval](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/3) 

# Handle

tensors


# Vulnerability details

## Impact
After swapping a yield source, the old yield source still has infinite approval. Infinite approval has been used in large attacks if the yield source isn't perfectly safe (see furucombo).

## Proof of Concept
https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L268

## Recommended Mitigation Steps
Decrease approval after swapping the yield source.

