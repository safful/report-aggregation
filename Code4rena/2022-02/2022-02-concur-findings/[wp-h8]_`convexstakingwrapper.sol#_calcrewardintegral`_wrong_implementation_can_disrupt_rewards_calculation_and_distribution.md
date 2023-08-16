## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H8] `ConvexStakingWrapper.sol#_calcRewardIntegral` Wrong implementation can disrupt rewards calculation and distribution](https://github.com/code-423n4/2022-02-concur-findings/issues/199) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConvexStakingWrapper.sol#L175-L204


# Vulnerability details

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConvexStakingWrapper.sol#L175-L204

```solidity
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
```

The problems in the current implementation:

- `reward.remaining` is not a global state; the `reward.remaining` of other `reward`s with the same rewardToken are not updated;
- `bal` should be refreshed before `reward.remaining = uint128(bal);`;
- L175 should not use `balanceOf` but take the diff before and after `getReward()`.

### PoC

- convexPool[1] is incentivized with CRV as the reward token, `1000 lpToken` can get `10 CRV` per day;
- convexPool[2] is incentivized with CRV as the reward token, `1000 lpToken` can get `20 CRV` per day.

1. Alice deposits `1,000` lpToken to `_pid` = `1`
2. 1 day later, Alice deposits `500` lpToken to `_pid` = `1` 

- convexPool `getReward()` sends `10 CRV` as reward to contract
- `d_reward` = 10, `2 CRV` sends to `treasury`, `8 CRV` send to `claimContract`
- `rewards[1][0].remaining` = 10

3. 0.5 day later, Alice deposits `500` lpToken to `_pid` = `1`, and the tx will fail:

- convexPool `getReward()` sends `7.5 CRV` as reward to contract
- `reward.remaining` = 10
- `bal` = 7.5
- `bal - reward.remaining` will fail due to underflow

4. 0.5 day later, Alice deposits `500` lpToken to `_pid` = `1`, most of the reward tokens will be left in the contract:

- convexPool `getReward()` sends `15 CRV` as reward to the contract;
- `d_reward = bal - reward.remaining` = 5
- `1 CRV` got sent to `treasury`, `4 CRV` sent to `claimContract`, `10 CRV` left in the contract;
- `rewards[1][0].remaining` = 15

Expected Results:

All the `15 CRV` get distributed: `3 CRV` to the `treasury`, and `12 CRV` to `claimContract`.

Actual Results:

Only `5 CRV` got distributed. The other `10 CRV` got left in the contract which can be frozen in the contract, see below for the details:

5. Bob deposits `1,000` lpToken to `_pid` = `2`

- convexPool `getReward()` sends `0 CRV` as reward to the contract
- `d_reward = bal - reward.remaining` = 10
- `2 CRV` sent to `treasury`, `8 CRV` sent to `claimContract` without calling `pushReward()`, so the `8 CRV` are now frozen in `claimContract`;
- `rewards[2][0].remaining` = 10

### Impact

- The two most important methods: `deposit()` and `withdraw()` will frequently fail as the tx will revert at `_calcRewardIntegral()`;
- Rewards distributed to users can often be fewer than expected;
- If there are different pools that use the same token as rewards, part of the rewards can be frozen at `claimContract` and no one can claim them.

### Recommendation

Consider comparing the `balanceOf` reward token before and after `getReward()` to get the actual rewarded amount, and `reward.remaining` should be removed.

