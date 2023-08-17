## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-11

# [Should Accrue Before Change, Loss of Rewards in case of change of settings](https://github.com/code-423n4/2023-01-reserve-findings/issues/287) 

# Lines of code

 https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/StRSR.sol#L820-L832


# Vulnerability details


In `StRSR.sol`, `_payoutRewards` is used to accrue the value of rewards based on the time that has passed since `payoutLastPaid`

Because of it's dependence on `totalStakes`, `stakeRate` and time, the function is rightfully called on every `stake` and `unstake`.

There is a specific instance, in which `_payoutRewards` should also be called, which could create either an unfair reward stream or a governance attack and that's when `setRewardPeriod` and `setRewardRatio` are called.

If you imagine the ratio at which rewards are paid out as a line, then you can see that by changing `rewardRatio` and `period` you're changing it's slope.

You should then agree, that while governance can *rightfully* change those settings, it should `_payoutRewards` first, to ensure that the slope of rewards changes only for rewards to be distributed after the setting has changed.

## Mitigation
Functions that change the slope or period size should accrue rewards up to that point.

This is to avoid:
- Incorrect reward distribution
- Change (positive or negative) of rewards from the past

Without accrual, the change will apply retroactively from `payoutLastPaid`

Which could:
- Change the period length prematurely
- Start a new period inadvertently
- Cause a gain or loss of yield to stakers

Instead of starting a new period

## Suggested refactoring

```solidity
function setRewardPeriod(uint48 val) public governance {
    require(val > 0 && val <= MAX_REWARD_PERIOD, "invalid rewardPeriod");
    _payoutRewards(); // @audit Payout rewards for fairness
    emit RewardPeriodSet(rewardPeriod, val);
    rewardPeriod = val;
    require(rewardPeriod * 2 <= unstakingDelay, "unstakingDelay/rewardPeriod incompatible");
}

function setRewardRatio(uint192 val) public governance {
    require(val <= MAX_REWARD_RATIO, "invalid rewardRatio");
    _payoutRewards(); // @audit Payout rewards for fairness
    emit RewardRatioSet(rewardRatio, val);
    rewardRatio = val;
}
```