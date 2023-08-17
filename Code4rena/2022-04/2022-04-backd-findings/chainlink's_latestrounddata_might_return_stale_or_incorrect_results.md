## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [Chainlink's latestRoundData might return stale or incorrect results](https://github.com/code-423n4/2022-04-backd-findings/issues/17) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/oracles/ChainlinkOracleProvider.sol#L55


# Vulnerability details

## Impact
On ChainlinkOracleProvider.sol and ChainlinkUsdWrapper.sol , we are using latestRoundData, but there is no check if the return value indicates stale data.
```
    function _ethPrice() private view returns (int256) {
        (, int256 answer, , , ) = _ethOracle.latestRoundData();
        return answer;
    }
...
    function getPriceUSD(address asset) public view override returns (uint256) {
        address feed = feeds[asset];
        require(feed != address(0), Error.ASSET_NOT_SUPPORTED);

        (, int256 answer, , uint256 updatedAt, ) = AggregatorV2V3Interface(feed).latestRoundData();

        require(block.timestamp <= updatedAt + stalePriceDelay, Error.STALE_PRICE);
        require(answer >= 0, Error.NEGATIVE_PRICE);

        uint256 price = uint256(answer);
        uint8 decimals = AggregatorV2V3Interface(feed).decimals();
        return price.scaleFrom(decimals);
    }
```
This could lead to stale prices according to the Chainlink documentation:

https://docs.chain.link/docs/historical-price-data/#historical-rounds
https://docs.chain.link/docs/faq/#how-can-i-check-if-the-answer-to-a-round-is-being-carried-over-from-a-previous-round
## Proof of Concept
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/oracles/ChainlinkOracleProvider.sol#L55
https://github.com/code-423n4/2022-04-backd/blob/main/backd/contracts/oracles/ChainlinkUsdWrapper.sol#L64
## Tools Used
None
## Recommended Mitigation Steps
```
    function _ethPrice() private view returns (int256) {
        (uint80 roundID, int256 answer, , uint256 timestamp, uint80 answeredInRound) = _ethOracle.latestRoundData();
        require(answeredInRound >= roundID, "Stale price");
        require(timestamp != 0,"Round not complete");
        require(answer > 0,"Chainlink answer reporting 0");
        return answer;
    }
...
    function getPriceUSD(address asset) public view override returns (uint256) {
        address feed = feeds[asset];
        require(feed != address(0), Error.ASSET_NOT_SUPPORTED);
        (uint80 roundID, int256 answer, , uint256 updatedAt, uint80 answeredInRound) = AggregatorV2V3Interface(feed).latestRoundData();
        require(answeredInRound >= roundID, "Stale price");
        require(answer > 0," Error.NEGATIVE_PRICE");
        require(block.timestamp <= updatedAt + stalePriceDelay, Error.STALE_PRICE);

        uint256 price = uint256(answer);
        uint8 decimals = AggregatorV2V3Interface(feed).decimals();
        return price.scaleFrom(decimals);
    }

