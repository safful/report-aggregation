## Tags

- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- MR-M-21

# [There is no way to retrieve the rewards from the `MultisigManager` and rewards are locked in the vault.](https://github.com/code-423n4/2023-02-gogopool-mitigation-contest-findings/issues/46) 

# Lines of code

https://github.com/multisig-labs/gogopool/blob/4bcef8b1d4e595c9ba41a091b2ebf1b45858f022/contracts/contract/RewardsPool.sol#L229


# Vulnerability details

# C4 issue

M-21: [Division by zero error can block RewardsPool#startRewardCycle if all multisig wallet are disabled.](https://github.com/code-423n4/2022-12-gogopool-findings/issues/143)

# Comments
The protocol provides an external function `startRewardsCycle()` so that anyone can start a new reward cycle if necessary.
Before mitigation, there was an edge case where this function will revert due to division by zero.
Edge case: there is no multisigs enabled. (possible when `Ocyticus.disableAllMultisigs(), Ocyticus.pauseEverything()` is called)

# Mitigation
[PR #37](https://github.com/multisig-labs/gogopool/pull/37)
If no multisig is enabled, the mitigation sends the rewards to the `MultisigManager` and it makes sense.
But this created another issue. There is no way to retrieve the rewards back from the `MultisigManager`.

# New issue
There is no way to retrieve the rewards from the `MultisigManager` and rewards are locked in the vault.

# Code snippet
https://github.com/multisig-labs/gogopool/blob/4bcef8b1d4e595c9ba41a091b2ebf1b45858f022/contracts/contract/RewardsPool.sol#L229

# Impact
There is no way to retrieve the rewards from the `MultisigManager` and rewards are locked in the vault.

# Proof of Concept
The rewards that were accrued in this specific edge case are locked in the `MultisigManager`.
It is understood that the funds are not lost and the protocol can be upgraded with a new `MultisigManager` contract with a proper function.
I evaluate the severity of the new issue as Medium because funds are locked in some specific edge cases and only withdrawable after contract upgrades.

# Tools used
Manual Review

# Recommended additional mitigation
Add a new external function in the `MultisigManager` with `guardianOrSpecificRegisteredContract("Ocyticus", msg.sender)` modifier and distribute the pending rewards to the active multisigs.

# Conclusion
Mitigation error - created another issue for the same edge case.

