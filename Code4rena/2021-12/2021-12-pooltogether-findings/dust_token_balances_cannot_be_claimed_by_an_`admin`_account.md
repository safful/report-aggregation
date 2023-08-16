## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Dust Token Balances Cannot Be Claimed By An `admin` Account](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/75) 

# Handle

leastwood


# Vulnerability details

## Impact

Users who have a small claim on rewards for various promotions, may not feasibly be able to claim these rewards as gas costs could outweigh the sum they receive in return. Hence, it is likely that a dust balance accrues overtime for tokens allocated for various promotions. Additionally, the `_calculateRewardAmount` calculation may result in truncated results, leading to further accrual of a dust balance. Therefore, it is useful that these funds do not go to waste.

## Proof of Concept

https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L162-L191

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider allowing an `admin` account to skim a promotion's tokens if it has been inactive for a certain length of time. There are several potential implementations, in varying degrees of complexity. However, the solution should attempt to maximise simplicity while minimising the accrual of dust balances.

