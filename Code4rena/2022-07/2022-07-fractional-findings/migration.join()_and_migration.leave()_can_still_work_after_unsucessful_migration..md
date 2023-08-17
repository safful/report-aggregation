## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Migration.join() and Migration.leave() can still work after unsucessful migration.](https://github.com/code-423n4/2022-07-fractional-findings/issues/250) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L105
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L141


# Vulnerability details

## Impact
Migration.join() and Migration.leave() can still work after unsucessful migration.
As I submitted with my high-risk finding "Migration.withdrawContribution() might work unexpectedly after unsuccessful migration.", withdraw logic after unsuccessful migration is different from the initial leave() logic and the withdrawal logic would be messy if users call join() and leave() after unsuccessful migration.


## Proof of Concept
According to the [explanation](https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L23), join() and leave() functions must be called for 7 days before commition.

Currently, such a scenario is possible.

- Alice creates a new migration and commits after some joins.
- The migration ended unsuccessfully after 4 days.
- Then users can call leave() or withdrawContribution() to withdraw their deposits but it wouldn't work properly because we should recalculate eth/fractional amounts with returned amounts after unsuccessful migration.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
We should add some restrictions to join() and leave() functions so that users can call these functions for 7 days before the migration is committed.

We should add these conditions to [join()](https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L118) and [leave()](https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L150).

```
require(!migrationInfo[_vault][_proposalId].isCommited, "committed already");
require(block.timestamp <= proposal.startTime + PROPOSAL_PERIOD, "proposal over");
```

