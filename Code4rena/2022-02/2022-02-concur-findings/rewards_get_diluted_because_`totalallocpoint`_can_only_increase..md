## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Rewards get diluted because `totalAllocPoint` can only increase.](https://github.com/code-423n4/2022-02-concur-findings/issues/221) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/MasterChef.sol


# Vulnerability details

## Impact
There is no functionality for removing pools/setting pool's allocPoints. Therefore `totalAllocPoint` only increases and rewards for pool decreases.

## Proof of Concept
Scenario:
1. Owner adds new pool (first pool) for staking with points = 900 (totalAllocPoint=900).
2. 1 week passes
3. First pool staking period ends (or for other reasons that pool is not meaningfully anymore).
4. Owner adds new pool (second pool) for staking with points = 100 (totalAllocPoint=1000)
5. 1 block later Alice stake 10 tokens there (at the same time).
6. 1 week passes
7. After some time Alice claims rewards. But she is eligible only for 10% of the rewards. 90% goes to unused pool.

## Tools Used
Manual review

## Recommended Mitigation Steps
Add functionality for removing pool or functionality for setting pool's `totalAllocPoint` param.

