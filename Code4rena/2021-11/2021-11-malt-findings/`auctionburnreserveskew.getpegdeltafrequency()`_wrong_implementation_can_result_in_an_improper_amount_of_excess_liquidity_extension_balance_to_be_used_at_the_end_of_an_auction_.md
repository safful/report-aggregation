## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [`AuctionBurnReserveSkew.getPegDeltaFrequency()` Wrong implementation can result in an improper amount of excess Liquidity Extension balance to be used at the end of an auction ](https://github.com/code-423n4/2021-11-malt-findings/issues/294) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionBurnReserveSkew.sol#L116-L132

```solidity=116{131}
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

When `count < auctionAverageLookback`, at L131, it should be `return total * 10000 / count;`. The current implementation will return a smaller value than expected.

The result of `getPegDeltaFrequency()` will be used for calculating `realBurnBudget` for auctions. With the result of `getPegDeltaFrequency()` being inaccurate, can result in an improper amount of excess Liquidity Extension balance to be used at the end of an auction.

