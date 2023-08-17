## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [In `Governance.sol`, it might be impossible to activate a new proposal forever after failed to execute the previous active proposal.](https://github.com/code-423n4/2022-08-olympus-findings/issues/376) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/policies/Governance.sol#L216-L221
https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/policies/Governance.sol#L302-L304


# Vulnerability details

## Impact
Currently, if users vote for the active proposal, the `VOTES` are transferred to the contract so that users can't vote or endorse other proposals while the voted proposal is active.

And the active proposal can be replaced only when the proposal is executed successfully or another proposal is activated after `GRACE_PERIOD`.

But `activateProposal()` requires at least 20% endorsements [here](https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/policies/Governance.sol#L216-L221), so it might be impossible to activate a new proposal forever if the current active proposal involves more than 80% of total votes.


## Proof of Concept
The below scenario would be possible.
1. `Proposal 1` was submitted and activated successfully.
2. Let's assume 81% of the total votes voted for this proposal. `Yes = 47%`, `No = 34%`
3. This proposal can't be executed for [this requirement](https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/policies/Governance.sol#L268-L270) because `47% - 34% = 13% < 33%`.
4. Currently the contract contains more than 81% of total votes and users have at most 19% in total.
5. Also users can't reclaim their votes among 81% while `Proposal 1` is active.
6. So even if a user who has 1% votes submits a new proposal, it's impossible to activate because of this [require()](https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/policies/Governance.sol#L216-L221).
7. So it's impossible to delete `Proposal 1` from an active proposal and there won't be other active proposal forever.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
I think we should add one more constant like `EXECUTION_EXPIRE = 2 weeks` so that voters can reclaim their votes after this period even if the proposal is active.

I am not sure we can use the current `GRACE_PERIOD` for that purpose.

So `reclaimVotes()` should be modified like below.

```
function reclaimVotes(uint256 proposalId_) external {
    uint256 userVotes = userVotesForProposal[proposalId_][msg.sender];

    if (userVotes == 0) {
        revert CannotReclaimZeroVotes();
    }

    if (proposalId_ == activeProposal.proposalId) {
        if (block.timestamp < activeProposal.activationTimestamp + EXECUTION_EXPIRE) //+++++++++++++++++++++++++++++++++
        {
            revert CannotReclaimTokensForActiveVote();
        }
    }

    if (tokenClaimsForProposal[proposalId_][msg.sender] == true) {
        revert VotingTokensAlreadyReclaimed();
    }

    tokenClaimsForProposal[proposalId_][msg.sender] = true;

    VOTES.transferFrom(address(this), msg.sender, userVotes);
}
```