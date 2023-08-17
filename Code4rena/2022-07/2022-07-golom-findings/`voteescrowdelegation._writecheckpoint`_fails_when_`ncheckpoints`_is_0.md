## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- old-submission-method
- selected-for-report

# [`VoteEscrowDelegation._writeCheckpoint` fails when `nCheckpoints` is 0](https://github.com/code-423n4/2022-07-golom-findings/issues/630) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L101
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L82-L86


# Vulnerability details

## Impact

When a user call `VoteEscrowDelegation.delegate` to make a delegation, it calls `VoteEscrowDelegation._writeCheckpoint` to update the checkpoint of `toTokenId`. However, if `nCheckpoints` is 0, `_writeCheckpoint` always reverts. What’s worse, `nCheckpoints` would be zero before any delegation has been made. In conclusion, users cannot make any delegation.

## Proof of Concept

When a user call `VoteEscrowDelegation.delegate` to make a delegation, it calls `VoteEscrowDelegation._writeCheckpoint` to update the checkpoint of `toTokenId`.
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L82-L86
```
    function delegate(uint256 tokenId, uint256 toTokenId) external {
        require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');
        require(this.balanceOfNFT(tokenId) >= MIN_VOTING_POWER_REQUIRED, 'VEDelegation: Need more voting power');

        delegates[tokenId] = toTokenId;
        uint256 nCheckpoints = numCheckpoints[toTokenId];

        if (nCheckpoints > 0) {
            Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
            checkpoint.delegatedTokenIds.push(tokenId);
            _writeCheckpoint(toTokenId, nCheckpoints, checkpoint.delegatedTokenIds);
        } else {
            uint256[] memory array = new uint256[](1);
            array[0] = tokenId;
            _writeCheckpoint(toTokenId, nCheckpoints, array);
        }

        emit DelegateChanged(tokenId, toTokenId, msg.sender);
    }
```

if `nCheckpoints` is 0, `_writeCheckpoint` always reverts.
Because `checkpoints[toTokenId][nCheckpoints - 1]` will trigger underflow in Solidity 0.8.11
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L101
```
    function _writeCheckpoint(
        uint256 toTokenId,
        uint256 nCheckpoints,
        uint256[] memory _delegatedTokenIds
    ) internal {
        require(_delegatedTokenIds.length < 500, 'VVDelegation: Cannot stake more');

        Checkpoint memory oldCheckpoint = checkpoints[toTokenId][nCheckpoints - 1];
        …
    }
```

## Tools Used

None

## Recommended Mitigation Steps

Fix `_writeCheckpoint`

```
    function _writeCheckpoint(
        uint256 toTokenId,
        uint256 nCheckpoints,
        uint256[] memory _delegatedTokenIds
    ) internal {
        require(_delegatedTokenIds.length < 500, 'VVDelegation: Cannot stake more');

   

        if (nCheckpoints > 0 && oldCheckpoint.fromBlock == block.number) {
            Checkpoint memory oldCheckpoint = checkpoints[toTokenId][nCheckpoints - 1];
            oldCheckpoint.delegatedTokenIds = _delegatedTokenIds;
        } else {
            checkpoints[toTokenId][nCheckpoints] = Checkpoint(block.number, _delegatedTokenIds);
            numCheckpoints[toTokenId] = nCheckpoints + 1;
        }
    }
```


