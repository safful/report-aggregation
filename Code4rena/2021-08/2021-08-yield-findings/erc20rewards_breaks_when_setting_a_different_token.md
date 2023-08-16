## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- ERC20Rewards

# [ERC20Rewards breaks when setting a different token](https://github.com/code-423n4/2021-08-yield-findings/issues/29) 

# Handle

cmichel


# Vulnerability details

The `setRewards` function allows setting a different token.
Holders of a previous reward period cannot all be paid out and will receive **their old reward amount** in the new token.

This leads to issues when the new token is more (less) valuable, or uses different decimals.

**Example:** Assume the first reward period paid out in `DAI` which has 18 decimals. Someone would have received `1.0 DAI = 1e18 DAI` if they called `claim` now. Instead, they wait until the new period starts with `USDC` (using only 6 decimals) and can `claim` their `1e18` reward amount in USDC which would equal `1e12 USDC`, one trillion USD.

## Impact
Changing the reward token only works if old and new tokens use the same decimals and have the exact same value. Otherwise, users that claim too late/early will lose out.

## Recommended Mitigation Steps
Disallow changing the reward token, or clear user's pending rewards of the old token. The second approach requires more code changes and keeping track of what token a user last claimed.

