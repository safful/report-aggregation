## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- sponsor acknowledged

# [`veCVXStrategy.manualRebalance` has wrong logic](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/47) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `veCVXStrategy.manualRebalance` function computes two ratios `currentLockRatio` and `newLockRatio` and compares them.

However, these ratios compute different things and are not comparable:
- `currentLockRatio = balanceInLock.mul(10**18).div(totalCVXBalance)` is a **percentage value** with 18 decimals (i.e. `1e18 = 100%`). Its max value can at most be `1e18`.
- `newLockRatio = totalCVXBalance.mul(toLock).div(MAX_BPS)` is a **CVX token amount**. It's unbounded and just depends on the `totalCVXBalance` amount.

The comparison that follows does not make sense:

```solidity
if (newLockRatio <= currentLockRatio) {
  // ...
}
```

## Impact
The rebalancing is broken and does not correctly rebalance.
It usually leads to locking nearly everything if `totalCVXBalance` is high.

## Recommended Mitigation Steps
Judging from the `cvxToLock = newLockRatio.sub(currentLockRatio)` it seems the desired computation is that the "ratios" should actually be in CVX amounts and not in percentages. Therefore, `currentLockRatio` should just be `balanceInLock`. (The variables should be renamed as they aren't really ratios but absolute CVX balance amounts.)


