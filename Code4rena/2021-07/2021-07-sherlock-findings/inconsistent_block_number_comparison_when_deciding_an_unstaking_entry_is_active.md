## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Inconsistent block number comparison when deciding an unstaking entry is active](https://github.com/code-423n4/2021-07-sherlock-findings/issues/139) 

# Handle

shw


# Vulnerability details

The `getInitialUnstakeEntry` function of `PoolBase` returns the first active unstaking entry of a staker, which requires the current block to be strictly before the last block in the unstaking window. However, the `unstake` function allows the current block to be exactly the same as the last block (same logic in `unstakeWindowExpiry`).

## Proof of Concept

Referenced code:
[PoolBase.sol#L136](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L136)
[PoolBase.sol#L344](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L344)
[PoolBase.sol#L364](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/PoolBase.sol#L364)

## Recommended Mitigation Steps

Change the `<=` comparison at line 136 to `<` for consistency.

