## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary dynamic size array `_averageTotalSupplies` can save gas](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/91) 

# Handle

WatchPug


# Vulnerability details

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L308-L323

```solidity=308{314,319,320}
uint64[] memory _epochStartTimestamps = new uint64[](1);
_epochStartTimestamps[0] = uint64(_epochStartTimestamp);

uint64[] memory _epochEndTimestamps = new uint64[](1);
_epochEndTimestamps[0] = uint64(_epochEndTimestamp);

uint256[] memory _averageTotalSupplies = _ticket.getAverageTotalSuppliesBetween(
    _epochStartTimestamps,
    _epochEndTimestamps
);

if (_averageTotalSupplies[0] > 0) {
    return (_promotion.tokensPerEpoch * _averageBalance) / _averageTotalSupplies[0];
}

return 0;
```

As there is only one time frame, `uint256[] memory _averageTotalSupplies = getAverageTotalSuppliesBetween(...)` can be changed to `uint256 _averageTotalSupply = getAverageTotalSuppliesBetween(...)[0]`, and `_averageTotalSupplies[0]` can be changed to `_averageTotalSupply` for gas saving.

