## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method
- selected-for-report

# [`VoteEscrowDelegation._transferFrom` can only be executed by the token owner](https://github.com/code-423n4/2022-07-golom-findings/issues/631) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L242
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L211


# Vulnerability details

## Impact

`VoteEscrowDelegation._transferFrom` should be successfully executed if `msg.sender` is the current owner, an authorized operator, or the approved address. `removeDelegation` is called in `_transferFrom`. `removeDelegation` only accepts the token owner. Thus, `_transferFrom` can only be executed by the token owner.

## Proof of Concept

`removeDelegation` is called in `_transferFrom`
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L242
```
    function _transferFrom(
        address _from,
        address _to,
        uint256 _tokenId,
        address _sender
    ) internal override {
        require(attachments[_tokenId] == 0 && !voted[_tokenId], 'attached');

        // remove the delegation
        this.removeDelegation(_tokenId);

        // Check requirements
        require(_isApprovedOrOwner(_sender, _tokenId));
        …
    }
```

However, `removeDelegation` only accept the token owner
https://github.com/code-423n4/2022-07-golom/blob/main/contracts/vote-escrow/VoteEscrowDelegation.sol#L211
```
    function removeDelegation(uint256 tokenId) external {
        require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');
        uint256 nCheckpoints = numCheckpoints[tokenId];
        Checkpoint storage checkpoint = checkpoints[tokenId][nCheckpoints - 1];
        removeElement(checkpoint.delegatedTokenIds, tokenId);
        _writeCheckpoint(tokenId, nCheckpoints, checkpoint.delegatedTokenIds);
    }
```

## Tools Used

None

## Recommended Mitigation Steps

Fix the permission control in `removeDelegation`


