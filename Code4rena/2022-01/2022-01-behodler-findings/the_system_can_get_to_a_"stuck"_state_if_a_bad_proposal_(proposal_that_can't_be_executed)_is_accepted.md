## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [The system can get to a "stuck" state if a bad proposal (proposal that can't be executed) is accepted](https://github.com/code-423n4/2022-01-behodler-findings/issues/153) 

# Handle

CertoraInc


# Vulnerability details

## LimboDAO.sol (`updateCurrentProposal()` modifier and `makeProposal()` function)
The LimboDAO contract has a variable that indicates the current proposal - every time there can be only one proposal. The only way a proposal can be done and a new proposal can be registered is to finish the previous proposal by either accepting it and executing it or by rejecting it. If a proposal that can't succeed, like for example an `UpdateMultipleSoulConfigProposal` proposal that has too much tokens and not enough gas, will stuck the system if it will be accepted. Thats because its time will pass - the users won't be able to vote anymore (because the `vote` function will revert), and the proposal can't be executed - the `execute` function will revert. So the proposal won't be able to be done and the system will be stuck because new proposal won't be able to be registered.

When trying to call the `executeCurrentProposal()` function that activates the `updateCurrentProposal()` modifier, the modifier will check the balance of fate, it will see that it's positive and will call `currentProposalState.proposal.orchestrateExecute()` to execute the proposal. the proposal will revert and cancel it all (leaving the proposal as the current proposal with `voting` state).

When trying to call `makeProposal()` function to make a new proposal it will revert because the current proposal is not equal to address(0).

To sum up, the system can get to a "stuck" state if a bad proposal (proposal that can't be executed) is accepted.

