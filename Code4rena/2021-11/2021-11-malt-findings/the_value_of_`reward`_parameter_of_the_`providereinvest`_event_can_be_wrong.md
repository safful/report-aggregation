## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The value of `reward` parameter of the `ProvideReinvest` event can be wrong](https://github.com/code-423n4/2021-11-malt-findings/issues/292) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardReinvestor.sol#L62-L76

```solidity=62
function provideReinvest(uint256 rewardLiquidity) external {
    _retrieveReward(rewardLiquidity);

    uint256 rewardBalance = rewardToken.balanceOf(address(this));

    // This is how much malt is required
    uint256 maltLiquidity = dexHandler.getOptimalLiquidity(address(malt), address(rewardToken), rewardBalance);

    // Transfer the remaining Malt required
    malt.safeTransferFrom(msg.sender, address(this), maltLiquidity);

    _bondAccount(msg.sender);

    emit ProvideReinvest(msg.sender, rewardLiquidity);
  }
```

`_retrieveReward` will call `MiningService.sol#withdrawRewardsForAccount()` which uses `amount` as max withdrawnAmount, if there are no enough rewards, the actual rewarded amount will be less than `amount`.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MiningService.sol#L155-L179

```solidity=155
function withdrawRewardsForAccount(address account, uint256 amount)
    public
    onlyRole(REINVESTOR_ROLE, "Must have reinvestor privs")
  {
    _withdrawMultiple(account, amount);
  }

  /*
   * INTERNAL FUNCTIONS
   */
  function _withdrawMultiple(address account, uint256 amount) internal {
    for (uint i = 0; i < mines.length; i = i + 1) {
      if (!mineActive[mines[i]]) {
        continue;
      }

      uint256 withdrawnAmount = IRewardMine(mines[i]).withdrawForAccount(account, amount, msg.sender);

      amount = amount.sub(withdrawnAmount);

      if (amount == 0) {
        break;
      }
    }
  }
```

### Recommendation

Consider using `rewardBalance` as the value of the `reward` parameter.

