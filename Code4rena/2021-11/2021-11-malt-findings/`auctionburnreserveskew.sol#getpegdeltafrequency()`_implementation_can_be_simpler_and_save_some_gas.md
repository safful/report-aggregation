## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`AuctionBurnReserveSkew.sol#getPegDeltaFrequency()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2021-11-malt-findings/issues/301) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L116-L132

```solidity=116
  function getPegDeltaFrequency() public view returns (uint256) {
    uint256 initialIndex = 0;
    uint256 index;

    if (count > auctionAverageLookback) {
      initialIndex = count - auctionAverageLookback;
    }

    uint256 total = 0;

    for (uint256 i = initialIndex; i < count; ++i) {
      index = _getIndexOfObservation(i);
      total = total + pegObservations[index];
    }

    return total * 10000 / auctionAverageLookback;
  }
```

### Recommendation

Change to:

```solidity=116
  function getPegDeltaFrequency() public view returns (uint256) {
    uint256 availablePegObservationsCount;
    {
      uint256 auctionAverageLookback_ = auctionAverageLookback;
      uint256 count_ = count;
      availablePegObservationsCount = count_ > auctionAverageLookback_ ? auctionAverageLookback_ : count_;
    }

    uint256 total = 0;
    for (uint256 i = 0; i < availablePegObservationsCount; ++i) {
      total += pegObservations[i];
    }
    return total * 10000 / availablePegObservationsCount;
  }
```

