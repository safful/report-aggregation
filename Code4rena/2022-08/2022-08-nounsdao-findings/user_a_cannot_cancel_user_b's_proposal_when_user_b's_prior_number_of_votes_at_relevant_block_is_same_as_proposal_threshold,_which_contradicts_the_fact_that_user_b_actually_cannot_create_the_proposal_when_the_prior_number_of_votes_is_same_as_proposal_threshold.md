## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User A cannot cancel User B's proposal when User B's prior number of votes at relevant block is same as proposal threshold, which contradicts the fact that User B actually cannot create the proposal when the prior number of votes is same as proposal threshold](https://github.com/code-423n4/2022-08-nounsdao-findings/issues/255) 

# Lines of code

https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L184-L279
https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L346-L368


# Vulnerability details

## Impact
When User B calls the following `propose` function for creating a proposal, it checks that User B's prior number of votes at the relevant block is larger than the proposal threshold through executing `nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold`. This means that User B cannot create the proposal when the prior number of votes and the proposal threshold are the same.

https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L184-L279
```solidity
    function propose(
        address[] memory targets,
        uint256[] memory values,
        string[] memory signatures,
        bytes[] memory calldatas,
        string memory description
    ) public returns (uint256) {
        ProposalTemp memory temp;

        temp.totalSupply = nouns.totalSupply();

        temp.proposalThreshold = bps2Uint(proposalThresholdBPS, temp.totalSupply);

        require(
            nouns.getPriorVotes(msg.sender, block.number - 1) > temp.proposalThreshold,
            'NounsDAO::propose: proposer votes below proposal threshold'
        );
        require(
            targets.length == values.length &&
                targets.length == signatures.length &&
                targets.length == calldatas.length,
            'NounsDAO::propose: proposal function information arity mismatch'
        );
        require(targets.length != 0, 'NounsDAO::propose: must provide actions');
        require(targets.length <= proposalMaxOperations, 'NounsDAO::propose: too many actions');

        temp.latestProposalId = latestProposalIds[msg.sender];
        if (temp.latestProposalId != 0) {
            ProposalState proposersLatestProposalState = state(temp.latestProposalId);
            require(
                proposersLatestProposalState != ProposalState.Active,
                'NounsDAO::propose: one live proposal per proposer, found an already active proposal'
            );
            require(
                proposersLatestProposalState != ProposalState.Pending,
                'NounsDAO::propose: one live proposal per proposer, found an already pending proposal'
            );
        }

        temp.startBlock = block.number + votingDelay;
        temp.endBlock = temp.startBlock + votingPeriod;

        proposalCount++;
        Proposal storage newProposal = _proposals[proposalCount];
        newProposal.id = proposalCount;
        newProposal.proposer = msg.sender;
        newProposal.proposalThreshold = temp.proposalThreshold;
        newProposal.eta = 0;
        newProposal.targets = targets;
        newProposal.values = values;
        newProposal.signatures = signatures;
        newProposal.calldatas = calldatas;
        newProposal.startBlock = temp.startBlock;
        newProposal.endBlock = temp.endBlock;
        newProposal.forVotes = 0;
        newProposal.againstVotes = 0;
        newProposal.abstainVotes = 0;
        newProposal.canceled = false;
        newProposal.executed = false;
        newProposal.vetoed = false;
        newProposal.totalSupply = temp.totalSupply;
        newProposal.creationBlock = block.number;

        latestProposalIds[newProposal.proposer] = newProposal.id;

        /// @notice Maintains backwards compatibility with GovernorBravo events
        emit ProposalCreated(
            newProposal.id,
            msg.sender,
            targets,
            values,
            signatures,
            calldatas,
            newProposal.startBlock,
            newProposal.endBlock,
            description
        );

        /// @notice Updated event with `proposalThreshold` and `minQuorumVotes`
        /// @notice `minQuorumVotes` is always zero since V2 introduces dynamic quorum with checkpoints
        emit ProposalCreatedWithRequirements(
            newProposal.id,
            msg.sender,
            targets,
            values,
            signatures,
            calldatas,
            newProposal.startBlock,
            newProposal.endBlock,
            newProposal.proposalThreshold,
            minQuorumVotes(),
            description
        );

        return newProposal.id;
    }
```

After User B's proposal is created, User A can call the following `cancel` function to cancel it. When calling `cancel`, it checks that User B's prior number of votes at the relevant block is less than the proposal threshold through executing `nouns.getPriorVotes(proposal.proposer, block.number - 1) < proposal.proposalThreshold`. When User B's prior number of votes and the proposal threshold are the same, User A cannot cancel this proposal of User B. However, this contradicts the fact User B actually cannot create this proposal when the same condition holds true. In other words, if User B cannot create this proposal when the prior number of votes and the proposal threshold are the same, User A should be able to cancel User B's proposal under the same condition but it is not true. The functionality for canceling User B's proposal in this situation becomes unavailable for User A.

https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L346-L368
```solidity
    function cancel(uint256 proposalId) external {
        require(state(proposalId) != ProposalState.Executed, 'NounsDAO::cancel: cannot cancel executed proposal');

        Proposal storage proposal = _proposals[proposalId];
        require(
            msg.sender == proposal.proposer ||
                nouns.getPriorVotes(proposal.proposer, block.number - 1) < proposal.proposalThreshold,
            'NounsDAO::cancel: proposer above threshold'
        );

        proposal.canceled = true;
        for (uint256 i = 0; i < proposal.targets.length; i++) {
            timelock.cancelTransaction(
                proposal.targets[i],
                proposal.values[i],
                proposal.signatures[i],
                proposal.calldatas[i],
                proposal.eta
            );
        }

        emit ProposalCanceled(proposalId);
    }
```

## Proof of Concept
Please append the following test in the `NounsDAOV2#inflationHandling` `describe` block in `test\governance\NounsDAO\V2\inflationHandling.test.ts`. This test should pass to demonstrate the described scenario.
```typescript
  it("User A cannot cancel User B's proposal when User B's prior number of votes at relevant block is same as proposal threshold, which contradicts the fact that User B actually cannot create the proposal when the prior number of votes is same as proposal threshold",
    async () => {
    // account1 has 3 tokens at the beginning
    // account1 gains 2 more to own 5 tokens in total
    await token.transferFrom(deployer.address, account1.address, 11);
    await token.transferFrom(deployer.address, account1.address, 12);

    await mineBlock();

    // account1 cannot create a proposal when owning 5 tokens in total
    await expect(
      gov.connect(account1).propose(targets, values, signatures, callDatas, 'do nothing'),
    ).to.be.revertedWith('NounsDAO::propose: proposer votes below proposal threshold');

    // account1 gains 1 more to own 6 tokens in total
    await token.transferFrom(deployer.address, account1.address, 13);

    await mineBlock();

    // account1 can create a proposal when owning 6 tokens in total
    await gov.connect(account1).propose(targets, values, signatures, callDatas, 'do nothing');
    const proposalId = await gov.latestProposalIds(account1.address);
    expect(await gov.state(proposalId)).to.equal(0);

    // other user cannot cancel account1's proposal at this moment
    await expect(
      gov.cancel(proposalId, {gasLimit: 1e6})
    ).to.be.revertedWith('NounsDAO::cancel: proposer above threshold');
    
    // account1 removes 1 token to own 5 tokens in total
    await token.connect(account1).transferFrom(account1.address, deployer.address, 13);

    await mineBlock();

    // other user still cannot cancel account1's proposal when account1 owns 5 tokens in total
    // this contradicts the fact that account1 cannot create a proposal when owning 5 tokens in total
    await expect(
      gov.cancel(proposalId, {gasLimit: 1e6})
    ).to.be.revertedWith('NounsDAO::cancel: proposer above threshold');

    // account1 removes another token to own 4 tokens in total
    await token.connect(account1).transferFrom(account1.address, deployer.address, 12);

    await mineBlock();

    // other user can now cancel account1's proposal when account1 owns 4 tokens in total
    await gov.cancel(proposalId, {gasLimit: 1e6})
    expect(await gov.state(proposalId)).to.equal(2);
  });
```


## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L197-L200 can be changed to the following code.
```solidity
        require(
            nouns.getPriorVotes(msg.sender, block.number - 1) >= temp.proposalThreshold,
            'NounsDAO::propose: proposer votes below proposal threshold'
        );
```
or

https://github.com/code-423n4/2022-08-nounsdao/blob/main/contracts/governance/NounsDAOLogicV2.sol#L350-L354 can be changed to the following code.
```solidity
        require(
            msg.sender == proposal.proposer ||
                nouns.getPriorVotes(proposal.proposer, block.number - 1) <= proposal.proposalThreshold,
            'NounsDAO::cancel: proposer above threshold'
        );
```

but not both.