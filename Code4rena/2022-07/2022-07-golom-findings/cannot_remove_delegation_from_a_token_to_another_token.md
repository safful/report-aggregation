## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [Cannot remove delegation from a token to another token](https://github.com/code-423n4/2022-07-golom-findings/issues/751) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L213


# Vulnerability details

## Impact
A user who has delegated the vote of a veGolom token (that he/she owns) to another veGolom token cannot remove the delegation, so the delegatee token will permanently hold the voting power of the delegator token.


## Proof of Concept
A user tries to remove the delegation from `tokenId` he/she owns to the delegated token, calling `removeDelegation(uint256 tokenId)`. 
The delegation should be removed at the lines:

```solidity
        Checkpoint storage checkpoint = checkpoints[tokenId][nCheckpoints - 1];
        removeElement(checkpoint.delegatedTokenIds, tokenId);
```
but the array `checkpoint.delegatedTokenIds` is the list of **delegators** to `tokenId` **itself**. So, unless the delegation was from the token to itself, `removeDelegation` does nothing.

## Recommended Mitigation Steps
Two fixes are proposed:

1. Add the delegatee as an argument to `removeDelegation` and remove `tokenId` from its list of delegators:
   
```diff
-   function removeDelegation(uint256 tokenId) external {
+   function removeDelegation(uint256 tokenId, uint256 toTokenId) external {
        require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');
        uint256 nCheckpoints = numCheckpoints[tokenId];
-       Checkpoint storage checkpoint = checkpoints[tokenId][nCheckpoints - 1];
+       Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
        removeElement(checkpoint.delegatedTokenIds, tokenId);
        _writeCheckpoint(tokenId, nCheckpoints, checkpoint.delegatedTokenIds);
    }
```

or

2. Load the delegatee from the mapping `delegates` which maps each delegator to its current delegatee:

```diff
    function removeDelegation(uint256 tokenId) external {
        require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');
+       uint256 toTokenId = delegates[tokenId];
        uint256 nCheckpoints = numCheckpoints[tokenId];
-       Checkpoint storage checkpoint = checkpoints[tokenId][nCheckpoints - 1];
+       Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
        removeElement(checkpoint.delegatedTokenIds, tokenId);
        _writeCheckpoint(tokenId, nCheckpoints, checkpoint.delegatedTokenIds);
    }
```