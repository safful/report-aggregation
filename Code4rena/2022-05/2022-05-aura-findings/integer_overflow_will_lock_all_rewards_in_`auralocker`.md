## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Integer overflow will lock all rewards in `AuraLocker`](https://github.com/code-423n4/2022-05-aura-findings/issues/261) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/contracts/AuraLocker.sol#L176-L177
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/contracts/AuraLocker.sol#L802-L814
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/contracts/AuraLocker.sol#L864


# Vulnerability details

## Impact

There is a potential overflow in the rewards calculations which would lead to `updateReward()` always reverting.

The impact of this overflow is that all reward tokens will be permanently locked in the contract. User's will be unable to call any of the functions which have the `updateReward()` modifier, that is:

- `lock()`
- `getReward()`
- `_processExpiredLocks()`
- `_notifyReward()`

As a result the contract will need to call `shutdown()` and the users will only be able to receive their staked tokens via `emergencyWithdraw()`, which does not transfer the users the reward tokens.

Note that if one reward token overflows this will cause a revert on all reward tokens due to the loop over reward tokens.

## Proof of Concept

The overflow may occur due to the base of values in `_rewardPerToken()`. 

```solidity
    function _rewardPerToken(address _rewardsToken) internal view returns (uint256) {
        if (lockedSupply == 0) {
            return rewardData[_rewardsToken].rewardPerTokenStored;
        }
        return
            uint256(rewardData[_rewardsToken].rewardPerTokenStored).add(
                _lastTimeRewardApplicable(rewardData[_rewardsToken].periodFinish)
                    .sub(rewardData[_rewardsToken].lastUpdateTime)
                    .mul(rewardData[_rewardsToken].rewardRate)
                    .mul(1e18)
                    .div(lockedSupply)
            );
    }
```

The return value of `_rewardPerToken()` is in terms of

```
(now - lastUpdateTime) * rewardRate * 10**18 / totalSupply
```

Here `(now - lastUpdateTime)` has a maximum value of `rewardDuration = 6 * 10**5`.

Now `rewardRate` is the `_reward.div(rewardsDuration)` as seen in `_notifyRewardAmount()` on line #864. Note that `rewardDuration` is a constant 604,800.

`rewardDuration = 6 * 10**5` 

Thus, if we have a rewards such as AURA or WETH (or most ERC20 tokens) which have units 10**18 we can transfer 1 WETH to the reward distributor which calls `_notifyRewardAmount()` and  sets the reward rate to,

`rewardRate = 10**18 / (6 * 10**5) ~= 10**12`

Finally, if this attack is run either by the first depositor they may `lock()` a single token which would set `totalSupply = 1`.

Therefore our equation in terms of units will become,
```
(now - lastUpdateTime) * rewardRate * 10**18 / totalSupply => 10**5 * 10**12 * 10**18 / 1 = 10**35
```

In since `rewardPerTokenStored` is a `uint96` it has a maximum value of `2**96 ~= 7.9 * 10**28`. Hence there will be an overflow in `newRewardPerToken.to96()`. Since we are unable to add more total supply due to `lock()` reverting there will be no way to circumvent this revert except to `shutdown()`.

```solidity
                uint256 newRewardPerToken = _rewardPerToken(token);
                rewardData[token].rewardPerTokenStored = newRewardPerToken.to96();
```

Note this attack is described when we have a low `totalSupply`. However it is also possible to apply this attack on a larger `totalSupply` when there are reward tokens which have decimal places larger than 18 or tokens which such as SHIB which have small token value and so many of the tokens can be bought for cheap.


## Recommended Mitigation Steps

To mitigation this issue it is recommended to increase the size of the `rewardPerTokenStored`. Since updating this value will require another slot to be used we recommend updating this to either `uint256` or to update both `rewardRate` and `rewardPerTokenStored` to be `uint224`.

