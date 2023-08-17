## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [`StrategyPUSDConvex.balanceOfJPEG` uses incorrect function signature while calling `extraReward.earned`, causing the function to unexpectedly revert everytime](https://github.com/code-423n4/2022-04-jpegd-findings/issues/139) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/vaults/yVault/strategies/StrategyPUSDConvex.sol#L234


# Vulnerability details

## Impact

As specified in Convex [BaseRewardPool.sol](https://github.com/convex-eth/platform/blob/main/contracts/contracts/BaseRewardPool.sol#L149) and [VirtualRewardPool.sol](https://github.com/convex-eth/platform/blob/main/contracts/contracts/VirtualBalanceRewardPool.sol#L127), the function signature of `earned` is `earned(address)`. However, `balanceOfJPEG` did not pass any arguments to `earned`, which would cause `balanceOfJPEG` to always revert.

This bug will propagate through `Controller` and `YVault` until finally reaching the source of the call in `YVaultLPFarming ._computeUpdate`, and render the entire farming contract unuseable.

## Proof of Concept

Both `BaseRewardPool.earned` and `VirtualBalanceRewardPool.earned` takes an address as argument

```
    function earned(address account) public view returns (uint256) {
        return
            balanceOf(account)
                .mul(rewardPerToken().sub(userRewardPerTokenPaid[account]))
                .div(1e18)
                .add(rewards[account]);
    }

    function earned(address account) public view returns (uint256) {
        return
            balanceOf(account)
                .mul(rewardPerToken().sub(userRewardPerTokenPaid[account]))
                .div(1e18)
                .add(rewards[account]);
    }
```

But `balanceOfJPEG` does not pass any address to `extraReward.earned`, causing the entire function to revert when called
```
    function balanceOfJPEG() external view returns (uint256) {
        uint256 availableBalance = jpeg.balanceOf(address(this));

        IBaseRewardPool baseRewardPool = convexConfig.baseRewardPool;
        uint256 length = baseRewardPool.extraRewardsLength();
        for (uint256 i = 0; i < length; i++) {
            IBaseRewardPool extraReward = IBaseRewardPool(baseRewardPool.extraRewards(i));
            if (address(jpeg) == extraReward.rewardToken()) {
                availableBalance += extraReward.earned();
                //we found jpeg, no need to continue the loop
                break;
            }
        }

        return availableBalance;
    }
```

## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

Pass `address(this)` as argument of `earned`.

Notice how we modify the fetching of reward. This is reported in a separate bug report, but for completeness, the entire fix is shown in both report entries.

```
    function balanceOfJPEG() external view returns (uint256) {
        uint256 availableBalance = jpeg.balanceOf(address(this));

        IBaseRewardPool baseRewardPool = convexConfig.baseRewardPool;
        availableBalance += baseRewardPool.earned(address(this));
        uint256 length = baseRewardPool.extraRewardsLength();
        for (uint256 i = 0; i < length; i++) {
            IBaseRewardPool extraReward = IBaseRewardPool(baseRewardPool.extraRewards(i));
            if (address(jpeg) == extraReward.rewardToken()) {
                availableBalance += extraReward.earned(address(this));
            }
        }

        return availableBalance;
    }
```


