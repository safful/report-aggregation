## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- edited-by-warden
- selected-for-report

# [When MIN_VOTING_POWER_REQUIRED is changed, previous votes are not affected.](https://github.com/code-423n4/2022-07-golom-findings/issues/626) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L168-L194


# Vulnerability details

## Impact
When MIN_VOTING_POWER_REQUIRED is changed, tokenIDs with votes lower than MIN_VOTING_POWER_REQUIRED will not be able to vote through the delegate function, but previous votes will not be affected.
Since MIN_VOTING_POWER_REQUIRED is mainly used to reduce the influence of spam users, changing this value should affect previous votes.
## Proof of Concept
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L168-L194
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L260-L262
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L73-L74
## Tools Used
None
## Recommended Mitigation Steps
In the getPriorVotes and getVotes functions, when the balance corresponding to tokenId is less than MIN_VOTING_POWER_REQUIRED, the value of votes will not be increased
```diff
    function getVotes(uint256 tokenId) external view returns (uint256) {
        uint256[] memory delegated = _getCurrentDelegated(tokenId);
        uint256 votes = 0;
        for (uint256 index = 0; index < delegated.length; index++) {
+         if(this.balanceOfNFT(delegated[index]) >= MIN_VOTING_POWER_REQUIRED){
            votes = votes + this.balanceOfNFT(delegated[index]);
+       }
        }
        return votes;
    }


    /**
     * @notice Determine the prior number of votes for an account as of a block number
     * @dev Block number must be a finalized block or else this function will revert to prevent misinformation.
     * @param tokenId The address of the account to check
     * @param blockNumber The block number to get the vote balance at
     * @return The number of votes the account had as of the given block
     */
    function getPriorVotes(uint256 tokenId, uint256 blockNumber) public view returns (uint256) {
        require(blockNumber < block.number, 'VEDelegation: not yet determined');
        uint256[] memory delegatednft = _getPriorDelegated(tokenId, blockNumber);
        uint256 votes = 0;
        for (uint256 index = 0; index < delegatednft.length; index++) {
+         if(this.balanceOfAtNFT(delegatednft[index], blockNumber) >= MIN_VOTING_POWER_REQUIRED){
            votes = votes + this.balanceOfAtNFT(delegatednft[index], blockNumber);
+         }
        }
        return votes;
    }
```