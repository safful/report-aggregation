## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Usage of deprecated ChainLink API in `Buoy3Pool`](https://github.com/code-423n4/2021-06-gro-findings/issues/106) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The Chainlink API (`latestAnswer`) used in the `Buoy3Pool` oracle wrappers is deprecated:

> This API is deprecated. Please see API Reference for the latest Price Feed API. [Chainlink Docs](https://docs.chain.link/docs/deprecated-aggregatorinterface-api-reference/#latestanswer)

## Impact
It seems like the old API can return stale data. Checks similar to that of the new API using `latestTimestamp` and `latestRoundare` are needed.
This could lead to stale prices according to the Chainlink documentation:
* [under current notifications: "if answeredInRound < roundId could indicate stale data."](https://docs.chain.link/docs/developer-communications#current-notifications)
* [under historical price data: "A timestamp with zero value means the round is not complete and should not be used."](https://docs.chain.link/docs/historical-price-data#solidity)

## Recommended Mitigation Steps
Add the recommended checks:
```solidity
(
    uint80 roundID,
    int256 price,
    ,
    uint256 timeStamp,
    uint80 answeredInRound
) = chainlink.latestRoundData();
require(
    timeStamp != 0,
    “ChainlinkOracle::getLatestAnswer: round is not complete”
);
require(
    answeredInRound >= roundID,
    “ChainlinkOracle::getLatestAnswer: stale data”
);
require(price != 0, "Chainlink Malfunction”);
```

