## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [_writeCheckpoint does not write to storage on same block](https://github.com/code-423n4/2022-07-golom-findings/issues/455) 

# Lines of code

https://github.com/golom-protocol/contracts/blob/4e84d5c2115d163ca54a1729da46164e8cf4df6d/contracts/vote-escrow/VoteEscrowDelegation.sol#L101-L108


# Vulnerability details

## Impact
In `VoteEscrowDelegation._writeCheckpoint`, when the checkpoint is overwritten in the same block the new value is set with `memory oldCheckpoint` and thus is never written to storage.

```javascript
Checkpoint memory oldCheckpoint = checkpoints[toTokenId][nCheckpoints - 1];

if (nCheckpoints > 0 && oldCheckpoint.fromBlock == block.number) {

oldCheckpoint.delegatedTokenIds = _delegatedTokenIds; 
}
```

Users that remove and delegate a token (or call `delegate` on the same token twice) in the same block will only have their first delegation persisted.

## Proof of Concept
- User delegates a `tokenId` by calling `delegate`.
- In the same block, the user decides to delgate the same token to a different token ID and calls `delegate` again which calls `_writeCheckpoint`.  Since this is the second transaction in the same block the if statement in the code block above executes and stores `_delegatedTokenIds` in `memory oldCheckpoint`, thus not persisting the array of `_delegatedTokenIds` in the checkpoint.

## Tools Used

Manual analysis

## Recommended Mitigation Steps
Define the `oldCheckpoint` variable as a `storage` pointer:

`Checkpoint storage oldCheckpoint = checkpoints[toTokenId][nCheckpoints - 1];`
