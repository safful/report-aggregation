## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [DAO proposals can be executed by anyone due to vulnerable TimelockController](https://github.com/code-423n4/2021-08-notional-findings/issues/58) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

The `GovernorAlpha` inherits from a vulnerable `TimelockController`.
This `TimelockController` allows an `EXECUTOR` role to escalate privileges and also gain the proposer role. See details on [OZ](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-fg47-3c2x-m2wr) and the [fix here](https://github.com/OpenZeppelin/openzeppelin-contracts/compare/v4.3.0...v4.3.1).
The bug is that `_executeBatch` checks if the proposal was scheduled only **after** the transactions have been executed. This allows inserting a call into the batch that schedules the batch itself, and the entire batch will succeed.
As the custom `GovernorAlpha.executeProposal` function removed the original "queued state check" (`require(state(proposalId) == ProposalState.Queued`), the attack can be executed by anyone, even without the `EXEUCTOR_ROLE`.

## POC
1. Create a proposal using `propose`. The calldata will be explained in the next step. (This can be done by anyone passing the min `proposalThreshold`)
2. Call `executeProposal(proposalId, ...)` such that the following calls are made:

```
call-0: grantRole(TIME_LOCK_ADMIN, attackerContract)
call-1: grantRole(EXECUTOR, attackerContract)
call-2: grantRole(PROPOSER, attackerContract)
call-3: updateDelay(0) // such that _afterCall "isOperationReady(id): timestamp[id] = block.timestamp + minDelay (0) <= block.timestamp" passes
call-4: attackerContract.hello() // this calls timelock.schedule(args=[targets, values, datas, ...]) where args were previously already stored in contract. (this is necessary because id depends on this function's args and we may not be self-referential)
// attackerContract is proposer & executor now and can directly call scheduleBatch & executeBatch without having to create a proposal
```

> ℹ️  I already talked to Jeff Wu about this and he created a test case for it confirming this finding

## Impact
Anyone who can create a proposal can become Timelock admin (proposer & executor) and execute arbitrary transactions as the DAO-controlled `GovernorAlpha`.
Note that this contract has severe privileges and an attacker can now do anything that previously required approval of the DAO. For example, they could update the `globalTransferOperator` and steal all tokens.

## Recommended Mitigation Steps
We recommend updating the vulnerable contract to `TimelockController v3.4.2`.
It currently uses `OpenZeppelin/openzeppelin-contracts@3.4.0-solc-0.7`


