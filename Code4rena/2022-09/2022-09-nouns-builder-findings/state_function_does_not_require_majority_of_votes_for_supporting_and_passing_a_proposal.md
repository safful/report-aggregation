## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [State function does not require majority of votes for supporting and passing a proposal](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/626) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L413-L456


# Vulnerability details

## Impact
When determining the proposal's state, the following `state` function is called, which can execute `else if (proposal.forVotes < proposal.againstVotes || proposal.forVotes < proposal.quorumVotes) { return ProposalState.Defeated; }`. If `proposal.forVotes` and `proposal.againstVotes` are the same, the proposal is not considered defeated when the quorum votes are reached by the for votes. However, many electoral systems require that the for votes to be more than the against votes in order to conclude that the proposal is passed because the majority of votes supports it. If the deployed DAO wants to require the majority of votes to support a proposal in order to pass it, the `state` function would incorrectly conclude that the proposal is not defeated when the for votes and against votes are the same at the end of voting. As a result, critical proposals, such as for updating implementations or withdrawing funds from the treasury, that should not be passed can be passed, or vice versa, so the impact can be huge.

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L413-L456
```solidity
    function state(bytes32 _proposalId) public view returns (ProposalState) {
        // Get a copy of the proposal
        Proposal memory proposal = proposals[_proposalId];

        // Ensure the proposal exists
        if (proposal.voteStart == 0) revert PROPOSAL_DOES_NOT_EXIST();

        // If the proposal was executed:
        if (proposal.executed) {
            return ProposalState.Executed;

            // Else if the proposal was canceled:
        } else if (proposal.canceled) {
            return ProposalState.Canceled;

            // Else if the proposal was vetoed:
        } else if (proposal.vetoed) {
            return ProposalState.Vetoed;

            // Else if voting has not started:
        } else if (block.timestamp < proposal.voteStart) {
            return ProposalState.Pending;

            // Else if voting has not ended:
        } else if (block.timestamp < proposal.voteEnd) {
            return ProposalState.Active;

            // Else if the proposal failed (outvoted OR didn't reach quorum):
        } else if (proposal.forVotes < proposal.againstVotes || proposal.forVotes < proposal.quorumVotes) {
            return ProposalState.Defeated;

            // Else if the proposal has not been queued:
        } else if (settings.treasury.timestamp(_proposalId) == 0) {
            return ProposalState.Succeeded;

            // Else if the proposal can no longer be executed:
        } else if (settings.treasury.isExpired(_proposalId)) {
            return ProposalState.Expired;

            // Else the proposal is queued
        } else {
            return ProposalState.Queued;
        }
    }
```

## Proof of Concept
Please append the following test in `test\Gov.t.sol`. This test will pass to demonstrate the described scenario.

```solidity
    function test_ProposalIsSucceededWhenNumberOfForAndAgainstVotesAreSame() public {
        vm.prank(founder);
        auction.unpause();

        createVoters(7, 5 ether);

        vm.prank(address(treasury));
        governor.updateQuorumThresholdBps(2000);

        bytes32 proposalId = createProposal();

        vm.warp(block.timestamp + governor.votingDelay());

        // number of for and against votes are both 2
        castVotes(proposalId, 2, 2, 3);

        vm.warp(block.timestamp + governor.votingPeriod());

        // the proposal is considered succeeded when number of for and against votes are the same after voting ends
        assertEq(uint256(governor.state(proposalId)), uint256(ProposalState.Succeeded));

        // the proposal can be queued afterwards
        governor.queue(proposalId);
        assertEq(uint256(governor.state(proposalId)), uint256(ProposalState.Queued));
    }
```

## Tools Used
VSCode

## Recommended Mitigation Steps
If there is no need to pass a proposal when `proposal.forVotes` and `proposal.againstVotes` are the same at the end of voting, then https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L441-L442 can be changed to the following code.
```solidity
        } else if (proposal.forVotes <= proposal.againstVotes || proposal.forVotes < proposal.quorumVotes) {
            return ProposalState.Defeated;
```

Otherwise, a governance configuration can be added to indicate whether the majority of votes is needed or not for supporting and passing a proposal. The `state` function then could return `ProposalState.Defeated` when `proposal.forVotes <= proposal.againstVotes` if so and when `proposal.forVotes < proposal.againstVotes` if not.