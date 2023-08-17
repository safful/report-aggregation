## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No check for existing extraRewards during push](https://github.com/code-423n4/2022-05-vetoken-findings/issues/89) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DRewardPool.sol#L138
ttps://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DLocker.sol#L156


# Vulnerability details

## Impact
Similar to a reported I submitted for BaseRewardPool.sol (https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/BaseRewardPool.sol#L126)

When adding `extraRewards` to the extra reward pool in https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DRewardPool.sol#L138 , there's no check for already existing address.
Assume a particular address takes up 2 slots out of 3, and a user withdraws staked extra rewards, the user will receive double the amount requested in https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DRewardPool.sol#L257-L258

## Proof of Concept
1.  Assume `rewardManager` had mistakenly added the same address twice in `addExtraReward()`
2. A user calls `stake()` , linked rewards is staked twice to the same address (unexpected behaviour I guess but not severe issue)
3. Now, user calls `withdraw()` to withdraw linked rewards (this is already 2x in step 2)
4. User will receive double the linked rewards due to the iteration in `https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DRewardPool.sol#L257-L258`

## Tools Used
Manual review

## Recommended Mitigation Steps
Guess a check for an already existing extraRewards can be added before Line 138

##Similar issue
**https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/VE3DLocker.sol#L156 - not so sure of the severity for this.
**https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/BaseRewardPool.sol#L126  - reported in a seperate report



