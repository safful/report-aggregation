## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ERC20Rewards

# [Gas optimization on `_updateRewardsPerToken` of `ERC20Rewards`](https://github.com/code-423n4/2021-08-yield-findings/issues/39) 

# Handle

shw


# Vulnerability details

## Impact

The `_updateRewardsPerToken` function of `ERC20Rewards` is called when a token is transferred, minted, or burned. Thus, it could be called multiple times in a single block. Gas optimization is possible by checking if the `end` variable is equal to `rewardsPerToken_.lastUpdated`. If so, the function can return after line 112 since no reward needs to be updated. The early return could avoid writing to the storage (line 117) and thus save gas.

## Proof of Concept

Referenced code:
[ERC20Rewards.sol#L112](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/utils/token/ERC20Rewards.sol#L112)
[ERC20Rewards.sol#L117](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/utils/token/ERC20Rewards.sol#L117)

## Recommended Mitigation Steps

Simply return if `end == rewardsPerToken_.lastUpdated` if the `_updateRewardsPerToken` function.

