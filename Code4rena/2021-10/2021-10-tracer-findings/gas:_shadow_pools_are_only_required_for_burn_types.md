## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: shadow pools are only required for burn types](https://github.com/code-423n4/2021-10-tracer-findings/issues/12) 

# Handle

cmichel


# Vulnerability details

The `PoolCommiter.shadowPools` track commitments for all four `Long/Short Mint/Burn` types and uses these to reconstruct the initial total supply to correctly compute the token amounts for the sequence of commitments (as short/long tokens already get burned in the commitment phase and reduced the total supply).
However, the two burn types `LongBurn` and `ShortBurn` are all that's needed for the reconstruction which can be seen from the fact that `shadowPools[.]` is only accessed with them.

#### Recommendation
Only store `shadowPools` for `LongBurn` and `ShortBurn` types, and remove the `shadowPools[_commitType] = shadowPools[_commitType] - _commit.amount;` statement in `_uncommit` which is unnecessary for the mints as it just pays out what's already tracked in the commitments (`_commit`).


