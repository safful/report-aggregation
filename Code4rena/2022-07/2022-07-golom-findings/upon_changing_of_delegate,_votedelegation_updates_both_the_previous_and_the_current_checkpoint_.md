## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- selected-for-report

# [Upon changing of delegate, VoteDelegation updates both the previous and the current checkpoint ](https://github.com/code-423n4/2022-07-golom-findings/issues/81) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L79
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L213


# Vulnerability details

The contract is accidently editing both the previous and current checkpoint when changing/removing a delegate.

## Impact
Incorrect counting of votes.

## Proof of Concept
If in `delegate` the delegate already has checkpoints, the function will grab the latest checkpoint, and add the `tokenId` to it. Note that it changes the storage variable.
```solidity
        if (nCheckpoints > 0) {
            Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
            checkpoint.delegatedTokenIds.push(tokenId);
            _writeCheckpoint(toTokenId, nCheckpoints, checkpoint.delegatedTokenIds);
```
It then calls `_writeCheckpoint`, which [will add](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L106) a new checkpoint if there's no checkpoint created for this block yet:
```solidity
        Checkpoint memory oldCheckpoint = checkpoints[toTokenId][nCheckpoints - 1];

        if (nCheckpoints > 0 && oldCheckpoint.fromBlock == block.number) {
            oldCheckpoint.delegatedTokenIds = _delegatedTokenIds;
        } else {
            checkpoints[toTokenId][nCheckpoints] = Checkpoint(block.number, _delegatedTokenIds);
            numCheckpoints[toTokenId] = nCheckpoints + 1;
        }
```
Therefore, if this function has created a new checkpoint with the passed `_delegatedTokenIds`, we already saw that the previous function has already added `tokenId` to the previous checkpoint, so now both the new checkpoint and the previous checkpoint will have `tokenId` in them.
This is wrong as it updates an earlier checkpoint with the latest change.

The same situation happens in [`removeDelegation`](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L213).

## Recommended Mitigation Steps
When reading the latest checkpoint:
```solidity
Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
```
Change the `storage` to `memory`. This way it will not affect the previous checkpoint, but will pass the correct updated array to `_writeCheckpoint`, which will then write/update the correct checkpoint.

