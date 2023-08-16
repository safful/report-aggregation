## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Chainlink's `latestRoundData` might return stale or incorrect results](https://github.com/code-423n4/2021-12-perennial-findings/issues/24) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L50-L60

```solidity=50
function sync() public {
    (, int256 feedPrice, , uint256 timestamp, ) = feed.latestRoundData();
    Fixed18 price = Fixed18Lib.ratio(feedPrice, SafeCast.toInt256(_decimalOffset));

    if (priceAtVersion.length == 0 || timestamp > timestampAtVersion[currentVersion()] + minDelay) {
        priceAtVersion.push(price);
        timestampAtVersion.push(timestamp);

        emit Version(currentVersion(), timestamp, price);
    }
}
```

On `ChainlinkOracle.sol`, we are using `latestRoundData`, but there is no check if the return value indicates stale data. This could lead to stale prices according to the Chainlink documentation:

- https://docs.chain.link/docs/historical-price-data/#historical-rounds
- https://docs.chain.link/docs/faq/#how-can-i-check-if-the-answer-to-a-round-is-being-carried-over-from-a-previous-round

### Recommendation

Consider adding missing checks for stale data. 

For example:

```solidity
(uint80 roundID, int256 feedPrice, , uint256 timestamp, uint80 answeredInRound) = feed.latestRoundData();
require(feedPrice > 0, "Chainlink price <= 0"); 
require(answeredInRound >= roundID, "Stale price");
require(timestamp != 0, "Round not complete");
```

