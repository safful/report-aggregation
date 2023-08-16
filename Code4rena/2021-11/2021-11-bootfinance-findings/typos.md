## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Typos](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/230) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1710-L1714

```solidity=1710
/**
     * @notice Stops ramping Target price immediately. Once this function is called, rampTargetPrce()
     * cannot be called for another 24 hours
     * @param self TargetPrice struct to update
     */
```

`rampTargetPrce` should be `rampTargetPrice`.

