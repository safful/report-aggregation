## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-14

# [RewardThrottle.populateFromPreviousThrottle may be exposed to front-run attack](https://github.com/code-423n4/2023-02-malt-findings/issues/8) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L660-L688


# Vulnerability details

## Impact
RewardThrottle.populateFromPreviousThrottle allows ADMIN_ROLE to use epochData from previousThrottle to populate state from activeEpoch to epoch in current RewardThrottle.
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
But since populateFromPreviousThrottle and _fillInEpochGaps have basically the same function, a malicious user can call fillInEpochGaps to front-run populateFromPreviousThrottle.
```solidity
  function _fillInEpochGaps(uint256 epoch) internal {
    uint256 epochsPerYear = timekeeper.epochsPerYear();
    uint256 _activeEpoch = activeEpoch; // gas

    state[_activeEpoch].bondedValue = bonding.averageBondedValue(_activeEpoch);
    state[_activeEpoch].epochsPerYear = epochsPerYear;
    state[_activeEpoch].desiredAPR = targetAPR;

    if (_activeEpoch > 0) {
      state[_activeEpoch].cumulativeCashflowApr =
        state[_activeEpoch - 1].cumulativeCashflowApr +
        epochCashflowAPR(_activeEpoch - 1);
      state[_activeEpoch].cumulativeApr =
        state[_activeEpoch - 1].cumulativeApr +
        epochAPR(_activeEpoch - 1);
    }

    // Avoid issues if gap between rewards is greater than one epoch
    for (uint256 i = _activeEpoch + 1; i <= epoch; ++i) {
      if (!state[i].active) {
        state[i].bondedValue = bonding.averageBondedValue(i);
        state[i].profit = 0;
        state[i].rewarded = 0;
        state[i].epochsPerYear = epochsPerYear;
        state[i].desiredAPR = targetAPR;
        state[i].cumulativeCashflowApr =
          state[i - 1].cumulativeCashflowApr +
          epochCashflowAPR(i - 1);
        state[i].cumulativeApr = state[i - 1].cumulativeApr + epochAPR(i - 1);
        state[i].active = true;
      }
    }

    activeEpoch = epoch;
  }
```
The only difference is that it seems that populateFromPreviousThrottle can make epoch and activeEpoch greater than timekeeper.epoch(), thereby updating the state for future epochs, but _fillInEpochGaps makes activeEpoch = timekeeper.epoch(), thereby invalidating populateFromPreviousThrottle for future updates. (This usage should be very unlikely)
## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/RewardSystem/RewardThrottle.sol#L660-L688
## Tools Used
None
## Recommended Mitigation Steps
If populateFromPreviousThrottle is used to initialize the state in the current RewardThrottle, it should be called on contract setup