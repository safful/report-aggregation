## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-06

# [Average `APR`s might be calculated wrongly after calling `populateFromPreviousThrottle()`.](https://github.com/code-423n4/2023-02-malt-findings/issues/30) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/RewardThrottle.sol#L660
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/RewardThrottle.sol#L139


# Vulnerability details

## Impact
Average `APR`s might be calculated wrongly after calling `populateFromPreviousThrottle()` and `targetAPR` might be changed unexpectedly.

## Proof of Concept
The epoch state struct contains `cumulativeCashflowApr` element and `cashflowAverageApr` is used to adjust `targetAPR` in `updateDesiredAPR()` function.

And `populateFromPreviousThrottle()` is an admin function to change `activeEpoch` and the relevant epoch state using the previous throttle.

And the `activeEpoch` is likely to be increased inside this function.

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

The problem might occur when `epoch < _activeEpoch + smoothingPeriod` because `state[epoch].cumulativeCashflowApr`and `state[epoch - smoothingPeriod].cumulativeCashflowApr` will be used for `cashflowAverageApr` calculation.

So `cumulativeCashflowApr` of the original epoch and the newly added epoch will be used together and `cashflowAverageApr` might be calculated wrongly.

As a result, `targetAPR` might be changed unexpectedly.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Recommend checking `epoch - _activeEpoch > smoothingPeriod` in `populateFromPreviousThrottle()`.