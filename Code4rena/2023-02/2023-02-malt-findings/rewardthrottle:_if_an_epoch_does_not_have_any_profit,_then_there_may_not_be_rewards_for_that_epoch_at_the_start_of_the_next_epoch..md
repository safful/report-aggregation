## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-02

# [RewardThrottle: If an epoch does not have any profit, then there may not be rewards for that epoch at the start of the next epoch.](https://github.com/code-423n4/2023-02-malt-findings/issues/20) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L437-L462


# Vulnerability details

## Impact
In RewardThrottle, both checkRewardUnderflow and fillInEpochGaps call _fillInEpochGaps to fill the state of the previous epoch without profit, the difference being that checkRewardUnderflow will request the reward from the overflowPool and distribute the reward, whereas fillInEpochGaps does not.
```solidity
  function checkRewardUnderflow() public onlyActive {
    uint256 epoch = timekeeper.epoch();

    uint256 _activeEpoch = activeEpoch; // gas

    // Fill in gaps so we have a fresh foundation to calculate from
    _fillInEpochGaps(epoch);

    if (epoch > _activeEpoch) {
      for (uint256 i = _activeEpoch; i < epoch; ++i) {
        uint256 underflow = _getRewardUnderflow(i);

        if (underflow > 0) {
          uint256 balance = overflowPool.requestCapital(underflow);

          _sendToDistributor(balance, i);
        }
      }
    }
  }

  function fillInEpochGaps() external {
    uint256 epoch = timekeeper.epoch();

    _fillInEpochGaps(epoch);
  }
```
This results in that when an epoch does not have any profit, then at the start of the next epoch that epoch will have a reward if checkRewardUnderflow is called, and no reward if fillInEpochGaps is called.
According to the documentation, when an epoch is not profitable enough, the reward should be requested from the overflowPool, so checkRewardUnderflow should be called. And if fillInEpochGaps is called first, the epoch will lose its reward.

Note: populateFromPreviousThrottle will also cause epochs without any profit to lose their rewards
```solidity
  function populateFromPreviousThrottle(address previousThrottle, uint256 epoch)
    external
    onlyRoleMalt(ADMIN_ROLE, "Only admin role")
  {
    RewardThrottle previous = RewardThrottle(previousThrottle);
    uint256 _activeEpoch = activeEpoch; // gas

    for (uint256 i = _activeEpoch; i < epoch; ++i) {
      (
        uint256 profit,
        uint256 rewarded,
        uint256 bondedValue,
        uint256 desiredAPR,
        uint256 epochsPerYear,
        uint256 cumulativeCashflowApr,
        uint256 cumulativeApr
      ) = previous.epochData(i);

      state[i].bondedValue = bondedValue;
      state[i].profit = profit;
      state[i].rewarded = rewarded;
      state[i].epochsPerYear = epochsPerYear;
      state[i].desiredAPR = desiredAPR;
      state[i].cumulativeCashflowApr = cumulativeCashflowApr;
      state[i].cumulativeApr = cumulativeApr;
    }

    activeEpoch = epoch;
  }
```
## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L437-L462
## Tools Used
None
## Recommended Mitigation Steps
Consider removing the fillInEpochGaps function, or only allowing it to be called when the contract is not active