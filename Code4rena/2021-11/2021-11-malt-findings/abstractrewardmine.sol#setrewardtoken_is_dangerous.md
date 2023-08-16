## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [AbstractRewardMine.sol#setRewardToken is dangerous](https://github.com/code-423n4/2021-11-malt-findings/issues/285) 

# Handle

0x0x0x


# Vulnerability details

## Impact

In case the reward token is changed, `totalDeclaredReward` will be changed and likely equal to `0`.  Since `_userStakePadding` and `_globalStakePadding` are accumulated, changing the reward token will not reset those values. Thus, it will create problems.

## Recommendation

I think it would be the best to remove this function. 

If you want to keep it, then it must have an event and it should be used by a timelock contract. Furthermore, it has to be used carefully and the new token should be distributed such that padding variables still make sense.

