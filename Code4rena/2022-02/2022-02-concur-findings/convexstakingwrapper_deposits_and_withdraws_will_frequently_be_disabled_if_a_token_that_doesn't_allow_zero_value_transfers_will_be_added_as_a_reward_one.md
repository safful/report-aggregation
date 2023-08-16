## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ConvexStakingWrapper deposits and withdraws will frequently be disabled if a token that doesn't allow zero value transfers will be added as a reward one](https://github.com/code-423n4/2022-02-concur-findings/issues/231) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L182


# Vulnerability details

## Impact

If deposits and withdraws are done frequently enough, the reward update operation they invoke will deal mostly with the case when there is nothing to add yet, i.e. `reward.remaining` match the reward token balance.

If reward token doesn't allow for zero value transfers, the reward update function will fail on an empty incremental reward transfer, which is now done unconditionally, reverting the caller deposit/withdrawal functionality

## Proof of Concept

When ConvexStakingWrapper isn't paused, every deposit and withdraw update current rewards via `_checkpoint` function before proceeding:

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L233

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L260

`_checkpoint` calls `_calcRewardIntegral` for each of the reward tokens of the pid:

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L220

`_calcRewardIntegral` updates the incremental reward for the token, running the logic even if reward is zero, which is frequently the case:

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L182

If the reward token doesn't allow zero value transfers, this transfer will fail, reverting the corresponding deposit or withdraw

## Recommended Mitigation Steps

Consider checking the reward before doing transfer (and the related computations as an efficiency measure):

Now:
```
IERC20(reward.token).transfer(address(claimContract), d_reward);
```

To be:
```
if (d_reward > 0)
	IERC20(reward.token).transfer(address(claimContract), d_reward);
```


