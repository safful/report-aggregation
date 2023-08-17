## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Chainlink's latestRoundData might return stale or incorrect results](https://github.com/code-423n4/2022-04-phuture-findings/issues/1) 

# Lines of code

https://github.com/code-423n4/2022-04-phuture/blob/main/contracts/ChainlinkPriceOracle.sol#L83-L84


# Vulnerability details

## Impact
On ChainlinkPriceOracle.sol, we are using latestRoundData, but there is no check if the return value indicates stale data.
```
        (, int basePrice, , , ) = baseAggregator.latestRoundData();
        (, int quotePrice, , , ) = assetInfo.aggregator.latestRoundData();
```
This could lead to stale prices according to the Chainlink documentation:

https://docs.chain.link/docs/historical-price-data/#historical-rounds
https://docs.chain.link/docs/faq/#how-can-i-check-if-the-answer-to-a-round-is-being-carried-over-from-a-previous-round
## Proof of Concept
https://github.com/code-423n4/2022-04-phuture/blob/main/contracts/ChainlinkPriceOracle.sol#L83-L84
## Tools Used
None
## Recommended Mitigation Steps
Consider adding missing checks for stale data.

For example:
```
    (uint80 baseRoundID, int256 basePrice, , uint256 baseTimestamp, uint80 BaseAnsweredInRound) = baseAggregator.latestRoundData();
    (uint80 quoteRoundID, int256 quotePrice, , uint256 quoteTimestamp, uint80 quoteAnsweredInRound) = assetInfo.aggregator.latestRoundData();
    require(BaseAnsweredInRound >= baseRoundID && quoteAnsweredInRound >=  quoteRoundID, "Stale price");
    require(baseTimestamp != 0 && quoteTimestamp != 0 ,"Round not complete");
    require(basePrice > 0 && quotePrice > 0,"Chainlink answer reporting 0");
```

