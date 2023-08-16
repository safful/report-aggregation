## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`ConvexStakingWrapper._calcRewardIntegral()` Can Be Manipulated To Steal Tokens From Other Pools](https://github.com/code-423n4/2022-02-concur-findings/issues/146) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L216-L259


# Vulnerability details

## Impact

The `ConvexStakingWrapper.sol` implementation makes several modifications to the original design. One of the key changes is the ability to add multiple pools into the wrapper contract, where each pool is represented by a unique `_pid`. By doing this, we are able to aggregate pools and their LP tokens to simplify the token distribution process. 

However, the interdependence between pools introduces new problems. Because the original implementation uses the contract's reward token balance to track newly claimed tokens, it is possible for a malicious user to abuse the unguarded `getReward` function to maximise the profit they are able to generate. By calling `getReward` on multiple pools with the same reward token (i.e. `cvx`), users are able to siphon rewards from other pools. This inevitably leads to certain loss of rewards for users who have deposited LP tokens into these victim pools. As `crv` and `cvx` are reward tokens by default, it is very likely that someone will want to exploit this issue.

## Proof of Concept

Let's consider the following scenario:
- There are two convex pools with `_pid` 0 and 1.
- Both pools currently only distribute `cvx` tokens.
- Alice deposits LP tokens into the pool with `_pid` 0.
- Both pools earn 100 `cvx` tokens which are to be distributed to the holders of the two pools.
- While Alice is a sole staker of the pool with `_pid` 0, the pool with `_pid` 1 has several stakers.
-  Alice decides she wants to maximise her potential rewards, so she directly calls the unguarded `IRewardStaking(convexPool[_pid]).getReward` function on both pools, resulting in 200 `cvx` tokens being sent to the contract.
- She then decides to deposit the 0 amount to execute the `_calcRewardIntegral` function on the pool with `_pid` 0. However, this function will calculate `d_reward` as `bal - reward.remaining` which is effectively the change in contract balance. As we have directly claimed `cvx` tokens over the two pools, this `d_reward` will be equal to 200.
- Alice is then entitled to the entire 200 tokens as she is the sole staker of her pool. So instead of receiving 100 tokens, she is able to siphon rewards from other pools.

Altogether, this will lead to the loss of rewards for other stakers as they are unable to then claim their rewards.

https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L216-L259
```
function _calcRewardIntegral(
    uint256 _pid,
    uint256 _index,
    address _account,
    uint256 _balance,
    uint256 _supply
) internal {
    RewardType memory reward = rewards[_pid][_index];

    //get difference in balance and remaining rewards
    //getReward is unguarded so we use remaining to keep track of how much was actually claimed
    uint256 bal = IERC20(reward.token).balanceOf(address(this));
    uint256 d_reward = bal - reward.remaining;
    // send 20 % of cvx / crv reward to treasury
    if (reward.token == cvx || reward.token == crv) {
        IERC20(reward.token).transfer(treasury, d_reward / 5);
        d_reward = (d_reward * 4) / 5;
    }
    IERC20(reward.token).transfer(address(claimContract), d_reward);

    if (_supply > 0 && d_reward > 0) {
        reward.integral =
            reward.integral +
            uint128((d_reward * 1e20) / _supply);
    }

    //update user integrals
    uint256 userI = userReward[_pid][_index][_account].integral;
    if (userI < reward.integral) {
        userReward[_pid][_index][_account].integral = reward.integral;
        claimContract.pushReward(
            _account,
            reward.token,
            (_balance * (reward.integral - userI)) / 1e20
        );
    }

    //update remaining reward here since balance could have changed if claiming
    if (bal != reward.remaining) {
        reward.remaining = uint128(bal);
    }

    rewards[_pid][_index] = reward;
}
```

## Tools Used

Manual code review.
Confirmation from Taek.

## Recommended Mitigation Steps

Consider redesigning this mechanism such that all pools have their `getReward` function called in `_checkpoint`. The `_calcRewardIntegral` function can then ensure that each pool is allocated only a fraction of the total rewards instead of the change in contract balance. Other implementations might be more ideal, so it is important that careful consideration is taken when making these changes.

