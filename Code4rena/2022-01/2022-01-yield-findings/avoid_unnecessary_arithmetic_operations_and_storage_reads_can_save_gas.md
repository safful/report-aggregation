## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary arithmetic operations and storage reads can save gas](https://github.com/code-423n4/2022-01-yield-findings/issues/101) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L106-L111

```solidity
    if (rewardsLength == 0) {
        RewardType storage reward = rewards.push();
        reward.reward_token = crv;
        reward.reward_pool = mainPool;
        rewardsLength += 1;
    }
```

When `rewardsLength` == `0`,  the new `rewardsLength` will always be 1. Therefore, replacing `+=` with `=` can avoid the unnecessary arithmetic operations and memory reads 


### Recommendation

Change to:

```solidity
    if (rewardsLength == 0) {
        RewardType storage reward = rewards.push();
        reward.reward_token = crv;
        reward.reward_pool = mainPool;
        rewardsLength = 1;
    }
```

