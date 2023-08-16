## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- fixed-in-upstream-repo

# [Staker.sol: Updating kValue requires interpolation with initial timestamp](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/69) 

# Handle

hickuphh3


# Vulnerability details

### Impact

Updating a `kValue` of a market requires interpolation against the initial timestamp, which can be a hassle and might lead to a wrong value set from what is expected. 

### Proof of Concept

Consider the following scenario:

- Initially set `kValue = 2e18`, `kPeriod = 2592000` (30 days)
- After 15 days, would like to refresh the market incentive (start again with `kValue = 2e18`), lasting another 30 days.

In the current implementation, the admin would call `_changeMarketLaunchIncentiveParameters()` with the following inputs:

- `period = 3888000` (45 days)
- `kValue` needs to be worked backwards from the formula

    `kInitialMultiplier - (((kInitialMultiplier - 1e18) * (block.timestamp - initialTimestamp)) / kPeriod)`. To achieve the desired effect, we would get `kValue = 25e17` (formula returns 2e18 after 15 days with kPeriod = 45 days).

This isn't immediately intuitive and could lead to mistakes.

### Recommended Mitigation Steps

Instead of calculating from `initialTimestamp` (when `addNewStakingFund()` was called), calculate from when the market incentives were last updated. This would require a new mapping to store last updated timestamps of market incentives.

For example, using the scenario above, refreshing the market incentive would mean using inputs `period = 2592000` (30 days) with `kValue = 2e18`.

```jsx
// marketIndex => timestamp of updated market launch incentive params 
mapping(uint32 => uint256) public marketLaunchIncentive_update_timestamps;

function _changeMarketLaunchIncentiveParameters(
  uint32 marketIndex,
  uint256 period,
  uint256 initialMultiplier
) internal virtual {
	require(initialMultiplier >= 1e18, "marketLaunchIncentiveMultiplier must be >= 1e18");

  marketLaunchIncentive_period[marketIndex] = period;
  marketLaunchIncentive_multipliers[marketIndex] = initialMultiplier;
	marketLaunchIncentive_update_timestamps[marketIndex] = block.timestamp;
};

function _getKValue(uint32 marketIndex) internal view virtual returns (uint256) {
  // Parameters controlling the float issuance multiplier.
  (uint256 kPeriod, uint256 kInitialMultiplier) = _getMarketLaunchIncentiveParameters(marketIndex);

  // Sanity check - under normal circumstances, the multipliers should
  // *never* be set to a value < 1e18, as there are guards against this.
  assert(kInitialMultiplier >= 1e18);

	// currently: uint256 initialTimestamp = accumulativeFloatPerSyntheticTokenSnapshots[marketIndex][0].timestamp;
	// changed to take from last updated timestamp instead of initial timestamp
  uint256 initialTimestamp = marketLaunchIncentive_update_timestamps[marketIndex];

  if (block.timestamp - initialTimestamp <= kPeriod) {
    return kInitialMultiplier - (((kInitialMultiplier - 1e18) * (block.timestamp - initialTimestamp)) / kPeriod);
  } else {
    return 1e18;
  }
}

```

