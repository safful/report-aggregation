## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [StakingRewards.setRewardsDuration allows setting near zero or enormous rewardsDuration, which breaks reward logic](https://github.com/code-423n4/2022-02-concur-findings/issues/223) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/StakingRewards.sol#L178-185


# Vulnerability details

## Impact

notifyRewardAmount will be inoperable if rewardsDuration bet set to zero. If will cease to produce meaningful results if rewardsDuration be too small or too big

## Proof of Concept

The setter do not control the value, allowing zero/near zero/enormous duration:

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/StakingRewards.sol#L178-185

Division by the duration is used in notifyRewardAmount:

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/StakingRewards.sol#L143-156

## Recommended Mitigation Steps

Check for min and max range in the rewardsDuration setter, as too small or too big rewardsDuration breaks the logic

