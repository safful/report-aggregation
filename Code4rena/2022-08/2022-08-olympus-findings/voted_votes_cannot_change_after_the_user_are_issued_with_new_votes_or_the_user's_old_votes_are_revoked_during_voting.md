## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Voted votes cannot change after the user are issued with new votes or the user's old votes are revoked during voting](https://github.com/code-423n4/2022-08-olympus-findings/issues/275) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Governance.sol#L240-L262
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/VoterRegistration.sol#L45-L48
https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/VoterRegistration.sol#L53-L56


# Vulnerability details

## Impact
A user can call the following `vote` function to vote for a proposal. During voting, the voter admin can still call the `issueVotesTo` and `revokeVotesFrom` functions below to issue new votes or revoke old votes for the user, which also changes the votes' total supply during the overall voting. Because each user can only call `vote` once for a proposal due to the `userVotesForProposal[activeProposal.proposalId][msg.sender] > 0` conditional check, the old voted votes, resulted from the `vote` call by the user, will be used to compare against the new total supply of the votes, resulted from the `issueVotesTo` and `revokeVotesFrom` calls during the overall voting, when determining whether the proposal can be executed or not. Because of this inconsistency, the result on whether the proposal can be executed might not be reliable.

https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/Governance.sol#L240-L262
```solidity
    function vote(bool for_) external {
        uint256 userVotes = VOTES.balanceOf(msg.sender);

        if (activeProposal.proposalId == 0) {
            revert NoActiveProposalDetected();
        }

        if (userVotesForProposal[activeProposal.proposalId][msg.sender] > 0) {
            revert UserAlreadyVoted();
        }

        if (for_) {
            yesVotesForProposal[activeProposal.proposalId] += userVotes;
        } else {
            noVotesForProposal[activeProposal.proposalId] += userVotes;
        }

        userVotesForProposal[activeProposal.proposalId][msg.sender] = userVotes;

        VOTES.transferFrom(msg.sender, address(this), userVotes);

        emit WalletVoted(activeProposal.proposalId, msg.sender, for_, userVotes);
    }
```

https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/VoterRegistration.sol#L45-L48
```solidity
    function issueVotesTo(address wallet_, uint256 amount_) external onlyRole("voter_admin") {
        // Issue the votes in the VOTES module
        VOTES.mintTo(wallet_, amount_);
    }
```

https://github.com/code-423n4/2022-08-olympus/blob/main/src/policies/VoterRegistration.sol#L53-L56
```solidity
    function revokeVotesFrom(address wallet_, uint256 amount_) external onlyRole("voter_admin") {
        // Revoke the votes in the VOTES module
        VOTES.burnFrom(wallet_, amount_);
    }
```

## Proof of Concept
Please add the following code in `src\test\policies\Governance.t.sol`.

First, please add the following code for `stdError`.
```solidity
import {Test, stdError} from "forge-std/Test.sol";    // @audit import stdError for testing purpose
```

Then, please append the following tests. These tests will pass to demonstrate the described scenarios.
```solidity
    function testScenario_UserCannotVoteAgainWithNewlyMintedVotes() public {
        _createActiveProposal();

        // voter3 votes for the proposal
        vm.prank(voter3);
        governance.vote(true);

        assertEq(governance.yesVotesForProposal(1), 300);
        assertEq(governance.noVotesForProposal(1), 0);

        assertEq(governance.userVotesForProposal(1, voter3), 300);
        assertEq(VOTES.balanceOf(voter3), 0);
        assertEq(VOTES.balanceOf(address(governance)), 300);

        // to simulate calling VoterRegistration.issueVotesTo that mints votes to voter3, VOTES.mintTo is called by godmode here
        vm.prank(godmode);
        VOTES.mintTo(voter3, 500);
        assertEq(VOTES.balanceOf(voter3), 500);

        // calling vote function again by voter3 reverts, which means that voter3 cannot additionally vote with the 500 newly minted votes
        vm.expectRevert(UserAlreadyVoted.selector);
        vm.prank(voter3);
        governance.vote(true);
    }
```

```solidity
    function testScenario_RevokeVotesAfterUserFinishsOwnVoting() public {
        _createActiveProposal();

        // voter3 votes for the proposal
        vm.prank(voter3);
        governance.vote(true);

        assertEq(governance.yesVotesForProposal(1), 300);
        assertEq(governance.noVotesForProposal(1), 0);

        assertEq(governance.userVotesForProposal(1, voter3), 300);
        assertEq(VOTES.balanceOf(voter3), 0);
        assertEq(VOTES.balanceOf(address(governance)), 300);

        // To simulate calling VoterRegistration.revokeVotesFrom that burns voter3's votes, VOTES.burnFrom is called by godmode here.
        // However, calling VOTES.burnFrom will revert due to arithmetic underflow.
        vm.prank(godmode);
        vm.expectRevert(stdError.arithmeticError);
        VOTES.burnFrom(voter3, 300);

        // the proposal is still voted with voter3's previous votes afterwards
        assertEq(governance.userVotesForProposal(1, voter3), 300);
        assertEq(VOTES.balanceOf(voter3), 0);
        assertEq(VOTES.balanceOf(address(governance)), 300);
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
When `issueVotesTo` and `revokeVotesFrom` are called during voting, the corresponding votes need to be added to or removed from the proposal's voted votes for the user. Alternatively, `issueVotesTo` and `revokeVotesFrom` can be disabled when an active proposal exists.