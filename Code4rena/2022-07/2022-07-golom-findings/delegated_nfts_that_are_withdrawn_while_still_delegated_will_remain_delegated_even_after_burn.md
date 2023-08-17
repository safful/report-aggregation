## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- edited-by-warden
- selected-for-report

# [Delegated NFTs that are withdrawn while still delegated will remain delegated even after burn](https://github.com/code-423n4/2022-07-golom-findings/issues/59) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowCore.sol#L1226-L1236


# Vulnerability details

## Impact
Burn NFTs remained delegated causing bloat and wasting gas

## Proof of Concept
VoteEscrowDelegation.sol doesn't change the withdraw or _burn functions inherited from VoteEscrowCore.sol. These functions are ignorant of the delegation system and don't properly remove the delegation when burning an NFT. The votes for the burned NFT will be removed but the reference will still be stored in the delegation list where it was last delegated. This creates a few issues. 1) It adds bloat to both getVotes and getPriorVotes because it adds a useless element that must be looped through. 2) The max number of users that can delegate to another NFT is 500 and the burned NFT takes up one of those spots reducing the number of real users that can delegate. 3) Adds gas cost when calling removeDelegation which adds gas cost to _transferFrom because removeElement has to cycle through a larger number of elements.

## Tools Used

## Recommended Mitigation Steps
Override _burn in VoteEscrowDelegation and add this.removeDelegation(_tokenId), similar to how it was done in _transferFrom

