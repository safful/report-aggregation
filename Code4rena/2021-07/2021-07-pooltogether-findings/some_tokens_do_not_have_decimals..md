## Tags

- bug
- 1 (Low Risk)
- SwappableYieldSource
- sponsor confirmed

# [Some tokens do not have decimals.](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/2) 

# Handle

tensors


# Vulnerability details

## Impact
There are a few tokens out there that do not use any decimals. As far as I know none of them would be a good yield source, but just in case something comes out, you may want to include the possibility that decimals = 0.

## Proof of Concept
https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L116

## Recommended Mitigation Steps
Remove the require statement. 

