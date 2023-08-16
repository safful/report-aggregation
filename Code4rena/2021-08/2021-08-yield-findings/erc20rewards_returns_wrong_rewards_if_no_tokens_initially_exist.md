## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- ERC20Rewards

# [ERC20Rewards returns wrong rewards if no tokens initially exist](https://github.com/code-423n4/2021-08-yield-findings/issues/28) 

# Handle

cmichel


# Vulnerability details

The `ERC20Rewards._updateRewardsPerToken` function exits without updating `rewardsPerToken_.lastUpdated` if `totalSupply` is zero, i.e., if there are no tokens initially.

This leads to an error if there is an active rewards period but not tokens have been minted yet.

**Example:** `rewardsPeriod.start: 1 month ago`, `rewardsPeriod.end: in 1 month`, `totalSupply == 0`.
The first mint leads to the user (mintee) receiving all rewards for the past period (50% of the total rewards in this case).
- `_mint` is called, calls `_updateRewardsPerToken` which short-circuits. `rewardsPerToken.lastUpdated` is still set to `rewardsPeriod.start` from the constructor. Then `_updateUserRewards` is called and does not currently yield any rewards. (because both balance and the index diff are zero). User is now minted the tokens, `totalSupply` increases and user balance is set.
- User performs a `claim`: `_updateRewardsPerToken` is called and `timeSinceLastUpdated = end - rewardsPerToken_.lastUpdated = block.timestamp - rewardsPeriod.start = 1 month`. Contract "issues" rewards for the past month. The first mintee receives all of it.

## Impact
The first mintee receives all pending rewards when they should not receive any past rewards.
This can easily happen if the token is new, the reward period has already been initialized and is running, but the protocol has not officially launched yet.
Note that `setRewards` also allows setting a date in the past which would also be fatal in this case.

## Recommended Mitigation Steps
The `rewardsPerToken_.lastUpdated` field must always be updated in `_updateRewardsPerToken` to the current time (or `end`) even if `_totalSupply == 0`. Don't return early.

