## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`++i` is more gas efficient than `i++`](https://github.com/code-423n4/2021-11-malt-findings/issues/295) 

# Handle

WatchPug


# Vulnerability details

Using `++i` is more gas efficient than `i++`, especially in a loop.

For example:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L54-L56
```solidity=54
for (uint i = 0; i < _period; i++) {
  pegObservations.push(0);
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L193-L195
```solidity=193
for (uint i = auctionAverageLookback; i < _lookback; i++) {
  pegObservations.push(0);
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L60-L62
```solidity=60
for (uint i = 0; i < sampleMemory; i++) {
  samples.push();
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L187-L195

```solidity=187
for (uint256 i = 0; i < sampleMemory; i++ ) {
  tempCount += 1;
  liveSample = samples[_getIndexOfSample(tempCount)];
  liveSample.timestamp = currentTimestamp;
  liveSample.cumulativeValue = currentCumulative;

  currentCumulative += addition;
  currentTimestamp += uint64(sampleLength);
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L292-L300
```solidity=292
for (uint256 i = 0; i < sampleMemory; i++ ) {
  tempCount += 1;
  liveSample = samples[_getIndexOfSample(tempCount)];
  liveSample.timestamp = currentTimestamp;
  liveSample.cumulativeValue = currentCumulative;

  currentCumulative += addition;
  currentTimestamp += uint64(sampleLength);
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L431-L433
```solidity=431
for (uint i = sampleMemory; i < _sampleMemory; i++) {
  samples.push();
}
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/libraries/UniswapV2Library.sol#L66-L69
```solidity=66
for (uint i; i < path.length - 1; i++) {
    (uint reserveIn, uint reserveOut) = getReserves(factory, path[i], path[i + 1]);
    amounts[i + 1] = getAmountOut(amounts[i], reserveIn, reserveOut);
}
```

