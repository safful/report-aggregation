## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Perform math inside code branch](https://github.com/code-423n4/2022-01-yield-findings/issues/106) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `_calcCvxIntegral()` function in ConvexStakingWrapper.sol doesn't use the same gas optimization that its sibling function `_calcRewardIntegral()` uses.

## Proof of Concept

This code is from [the `_calcCvxIntegral()` function](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L154)
```
if (_isClaim || userI < cvxRewardIntegral) {
    uint256 receiveable = cvx_claimable_reward[_accounts[u]] +
        ((_balances[u] * (cvxRewardIntegral - userI)) / 1e20);
    if (_isClaim) {
        if (receiveable > 0) {
            cvx_claimable_reward[_accounts[u]] = 0;
            IERC20(cvx).safeTransfer(_accounts[u], receiveable);
            bal = bal - (receiveable);
        }
    } else {
        cvx_claimable_reward[_accounts[u]] = receiveable;
    }
    cvx_reward_integral_for[_accounts[u]] = cvxRewardIntegral;
}
```

The related code from [the `_calcRewardIntegral()` function](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L206) has the receivable calculation inside the `if (_isClaim)` code branch to save gas if _isClaim is false.

```
if (_isClaim || userI < rewardIntegral) {
    if (_isClaim) {
        uint256 receiveable = reward.claimable_reward[_accounts[u]] +
            ((_balances[u] * (uint256(rewardIntegral) - userI)) / 1e20);
        if (receiveable > 0) {
            reward.claimable_reward[_accounts[u]] = 0;
            IERC20(reward.reward_token).safeTransfer(_accounts[u], receiveable);
            bal = bal - receiveable;
        }
    } else {
        reward.claimable_reward[_accounts[u]] =
            reward.claimable_reward[_accounts[u]] +
            ((_balances[u] * (uint256(rewardIntegral) - userI)) / 1e20);
    }
    reward.reward_integral_for[_accounts[u]] = rewardIntegral;
}
```

This optimization would save gas each time `_checkpoint()` is called because `_checkpoint()` sets _isClaim to false and doesn't enter the `if(_isClaim)` branch. 

## Recommended Mitigation Steps

Modify the `_calcCvxIntegral()` function to place the receiveable calculation inside the `if (_isClaim)` code branch.

