## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed
- ERC20Rewards

# [ERC20Rewards.sol: Consider making rewardsToken immutable](https://github.com/code-423n4/2021-08-yield-findings/issues/56) 

# Handle

hickuphh3


# Vulnerability details

### Impact

While it might seem like a good feature to have, being able to switch reward tokens will only be useful for tokens which are equivalent in value (probably stablecoins, pegged tokens) since it carries over unclaimed rewards from the previous reward program. It would be safer to keep the reward token immutable as a safeguard against violations of this condition.

