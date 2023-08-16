## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Rewards distribution can be disrupted by a early user](https://github.com/code-423n4/2022-01-yield-findings/issues/116) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/ConvexStakingWrapper.sol#L206-L224

```solidity
function _calcRewardIntegral(
    uint256 _index,
    address[2] memory _accounts,
    uint256[2] memory _balances,
    uint256 _supply,
    bool _isClaim
) internal {
    RewardType storage reward = rewards[_index];

    uint256 rewardIntegral = reward.reward_integral;
    uint256 rewardRemaining = reward.reward_remaining;

    //get difference in balance and remaining rewards
    //getReward is unguarded so we use reward_remaining to keep track of how much was actually claimed
    uint256 bal = IERC20(reward.reward_token).balanceOf(address(this));
    if (_supply > 0 && (bal - rewardRemaining) > 0) {
        rewardIntegral = uint128(rewardIntegral) + uint128(((bal - rewardRemaining) * 1e20) / _supply);
        reward.reward_integral = uint128(rewardIntegral);
    }
```

`reward.reward_integral` is `uint128`, if a early user mint (wrap) just `1` Wei of `convexToken`, and make `_supply == 1`, and then tranferring `5e18` of `reward_token` to the contract.

As a result, `reward.reward_integral` can exceed `type(uint128).max` and overflow, causing the rewards distribution to be disrupted.

### Recommendation

Consider `wrap` a certain amount of initial totalSupply, e.g. `1e8`, and never burn it. And consider using uint256 instead of uint128 for `reward.reward_integral`. Also, consdier lower `1e20` down to `1e12`.

