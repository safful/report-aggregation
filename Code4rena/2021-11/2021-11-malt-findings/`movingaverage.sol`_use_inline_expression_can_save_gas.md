## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`MovingAverage.sol` Use inline expression can save gas](https://github.com/code-423n4/2021-11-malt-findings/issues/269) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L356-L360

```solidity=356
  function _getCurrentSample() private view returns (Sample storage currentSample) {
    // Active sample is always counter - 1. Counter is the in progress sample
    uint32 currentSampleIndex = _getIndexOfSample(counter - 1);
    currentSample = samples[currentSampleIndex];
  }
```

The local variable `currentSampleIndex` is used only once. Making the expression inline can save gas.

Similar issue exists in `_getFirstSample()`, `_getNthSample()`, `AuctionBurnReserveSkew.sol#getRealBurnBudget()`, `MovingAverage.sol#_getFirstSample()`.

### Recommendation

Change to:

```solidity=356
  function _getCurrentSample() private view returns (Sample storage currentSample) {
    // Active sample is always counter - 1. Counter is the in progress sample
    currentSample = samples[_getIndexOfSample(counter - 1)];
  }
```

