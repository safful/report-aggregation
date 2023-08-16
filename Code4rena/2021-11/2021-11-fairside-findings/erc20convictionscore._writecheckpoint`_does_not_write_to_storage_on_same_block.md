## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ERC20ConvictionScore._writeCheckpoint` does not write to storage on same block](https://github.com/code-423n4/2021-11-fairside-findings/issues/69) 

# Handle

cmichel


# Vulnerability details

In `ERC20ConvictionScore._writeCheckpoint`, when the checkpoint is overwritten (`checkpoint.fromBlock == blockNumber`), the new value is set to the `memory checkpoint` structure and never written to storage.

```solidity
// @audit this is MEMORY, setting new convictionScore doesn't write to storage
Checkpoint memory checkpoint = checkpoints[user][nCheckpoints - 1];

if (nCheckpoints > 0 && checkpoint.fromBlock == blockNumber) {
    checkpoint.convictionScore = newCS;
}
```

Users that have their conviction score updated several times in the same block will only have their first score persisted.

#### POC
- User updates their conviction with `updateConvictionScore(user)`
- **In the same block**, the user now redeems an NFT conviction using `acquireConviction(id)`. This calls `_increaseConvictionScore(user, amount)` which calls `_writeCheckpoint(..., prevConvictionScore + amount)`. The updated checkpoint is **not** written to storage, and the user lost their conviction NFT. (The conviction/governance totals might still be updated though, leading to a discrepancy.)

## Impact
Users that have their conviction score updated several times in the same block will only have their first score persisted.

This also applies to the total conviction scores `TOTAL_CONVICTION_SCORE` and `TOTAL_GOVERNANCE_SCORE` (see `_updateConvictionTotals`) which is a big issue as these are updated a lot of times each block.

It can also be used for inflating a user's conviction by first calling `updateConvictionScore` and then creating conviction tokens with `tokenizeConviction`. The `_resetConviction` will not actually reset the user's conviction.

## Recommended Mitigation Steps
Define the `checkpoint` variable as a `storage` pointer:

```solidity
Checkpoint storage checkpoint = checkpoints[user][nCheckpoints - 1];
```

