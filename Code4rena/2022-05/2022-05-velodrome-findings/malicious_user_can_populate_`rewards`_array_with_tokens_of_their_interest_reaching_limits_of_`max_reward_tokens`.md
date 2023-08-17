## Tags

- bug
- 2 (Med Risk)
- sponsor acknowledged
- sponsor confirmed

# [Malicious user can populate `rewards` array with tokens of their interest reaching limits of `MAX_REWARD_TOKENS`](https://github.com/code-423n4/2022-05-velodrome-findings/issues/182) 

# Lines of code

https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Bribe.sol#L41-L60


# Vulnerability details

## Impact

Malicious user can populate `rewards` array with different tokens early reaching limit of `MAX_REWARD_TOKENS` sending very small amount of different tokens. It will restrict any other tokens to be used as `rewards` in [Bribe.sol#notifyRewardAmount()](https://github.com/code-423n4/2022-05-velodrome/blob/7fda97c570b758bbfa7dd6724a336c43d4041740/contracts/contracts/Bribe.sol#L41)

## Proof of Concept

A custom malicious contract can be created that can make multiple calls to `notifyRewardAmount()` sending very small amounts of different tokens to populate the array `rewards` and fulfill the total of `MAX_REWARD_TOKENS` . This will restrict any other person from adding to `rewards` array.


## Tools Used

- Manual analysis

## Recommended Mitigation Steps

