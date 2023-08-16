## Tags

- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Users Can Bypass Emergency Restrictions on updateUserRewardState()](https://github.com/code-423n4/2022-03-paladin-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1338-L1378

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L876-L906

# Vulnerability details

## Impact

The `emergencyWithdraw()` function intends to withdraw their tokens regardless if they are locked up for any duration. This emergency must be triggered by the owner of the contract by calling `triggerEmergencyWithdraw()`. A number of functions will revert when the protocol is in an emergency state, including all stake, lock, unlock and kick actions and the updating of a user's rewards. However, a user could bypass the restriction on `_updateUserRewards()` by transferring a small amount of unlocked tokens to their account. `_beforeTokenTransfer()` will call `_updateUserRewards()` on the `from` and `to` accounts. As a result, users can continue to accrue rewards while the protocol is in an emergency state and it makes sense for users to delay their emergency withdraw as they will be able to claim a higher proportion of the allocated rewards.

## Recommended Mitigation Steps

Consider adding a check for the boolean `emergency` value in `_beforeTokenTransfer()` to not call `_updateUserRewards` on any account if this value is set. Alternatively, a check could be added into the `_updateUserRewards()` function to return if `emergency` is true.
