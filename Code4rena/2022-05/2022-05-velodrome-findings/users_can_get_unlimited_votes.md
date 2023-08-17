## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Users can get unlimited votes](https://github.com/code-423n4/2022-05-velodrome-findings/issues/129) 

# Lines of code

https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L517-L528


# Vulnerability details

## Impact
Users can get unlimited votes which leads to them:
1. gaining control over governance
2. getting undeserved rewards
3. having their pools favored due to gauge values

## Proof of Concept
`_mint()` calls `_moveTokenDelegates()` to set up delegation...
```solidity
File: contracts/contracts/VotingEscrow.sol   #1

462       function _mint(address _to, uint _tokenId) internal returns (bool) {
463           // Throws if `_to` is zero address
464           assert(_to != address(0));
465           // TODO add delegates
466           // checkpoint for gov
467           _moveTokenDelegates(address(0), delegates(_to), _tokenId);
```
https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L462-L467

and `_transferFrom()` calls `_moveTokenDelegates()` to transfer delegates...
```solidity
File: contracts/contracts/VotingEscrow.sol   #2

301       function _transferFrom(
302           address _from,
303           address _to,
304           uint _tokenId,
305           address _sender
306       ) internal {
307           require(attachments[_tokenId] == 0 && !voted[_tokenId], "attached");
308           // Check requirements
309           require(_isApprovedOrOwner(_sender, _tokenId));
310           // Clear approval. Throws if `_from` is not the current owner
311           _clearApproval(_from, _tokenId);
312           // Remove NFT. Throws if `_tokenId` is not a valid NFT
313           _removeTokenFrom(_from, _tokenId);
314           // TODO delegates
315           // auto re-delegate
316           _moveTokenDelegates(delegates(_from), delegates(_to), _tokenId);
```
https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L301-L316

but `_burn()` does not transfer them back to `address(0)`
```solidity
File: contracts/contracts/VotingEscrow.sol   #3

517       function _burn(uint _tokenId) internal {
518           require(_isApprovedOrOwner(msg.sender, _tokenId), "caller is not owner nor approved");
519   
520           address owner = ownerOf(_tokenId);
521   
522           // Clear approval
523           approve(address(0), _tokenId);
524           // TODO add delegates
525           // Remove token
526           _removeTokenFrom(msg.sender, _tokenId);
527           emit Transfer(owner, address(0), _tokenId);
528       }
```
https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/VotingEscrow.sol#L517-L528

A user can deposit a token, lock it, wait for the lock to expire, transfer the token to another address, and repeat. During each iteration, a new NFT is minted and checkpointed. Calls to `getPastVotes()` will show the wrong values, since it will think the account still holds the delegation of the burnt NFT. Bribes and gauges also look at the checkpoints and will also have the wrong information

## Tools Used
Code inspection

## Recommended Mitigation Steps
Call `_moveTokenDelegates(owner,address(0))` in `_burn()`


