## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [[WP-H1] Rewards distribution can be disrupted by a early user](https://github.com/code-423n4/2022-02-concur-findings/issues/193) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/02d286253cd5570d4e595527618366f77627cdaf/contracts/ConvexStakingWrapper.sol#L184-L188


# Vulnerability details

https://github.com/code-423n4/2022-02-concur/blob/02d286253cd5570d4e595527618366f77627cdaf/contracts/ConvexStakingWrapper.sol#L184-L188

```solidity
if (_supply > 0 && d_reward > 0) {
    reward.integral =
        reward.integral +
        uint128((d_reward * 1e20) / _supply);
}
```

`reward.integral` is `uint128`, if an early user deposits with just `1` Wei of `lpToken`, and make `_supply == 1`, and then transferring `5e18` of `reward_token` to the contract.

As a result, `reward.integral` can exceed `type(uint128).max` and overflow, causing the rewards distribution to be disrupted.

### Recommendation

Consider `wrap` a certain amount of initial totalSupply at deployment, e.g. `1e8`, and never burn it. And consider using uint256 instead of uint128 for `reward.integral`. Also, consdier lower `1e20` down to `1e12`.

