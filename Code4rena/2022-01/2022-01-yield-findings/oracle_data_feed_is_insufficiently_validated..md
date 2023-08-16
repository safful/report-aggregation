## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Oracle data feed is insufficiently validated.](https://github.com/code-423n4/2022-01-yield-findings/issues/136) 

# Handle

throttle


# Vulnerability details

## Impact
Price can be stale and can lead to wrong `quoteAmount` return value

## Proof of Concept
Oracle data feed is insufficiently validated. There is no check for stale price and round completeness.
Price can be stale and can lead to wrong `quoteAmount` return value
```javascript
function _peek(
    bytes6 base,
    bytes6 quote,
    uint256 baseAmount
) private view returns (uint256 quoteAmount, uint256 updateTime) {
    ...

    (, int256 daiPrice, , , ) = DAI.latestRoundData();
    (, int256 usdcPrice, , , ) = USDC.latestRoundData();
    (, int256 usdtPrice, , , ) = USDT.latestRoundData();

    require(
        daiPrice > 0 && usdcPrice > 0 && usdtPrice > 0,
        "Chainlink pricefeed reporting 0"
    );

    ...
}
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Validate data feed
```javascript
function _peek(
    bytes6 base,
    bytes6 quote,
    uint256 baseAmount
) private view returns (uint256 quoteAmount, uint256 updateTime) {
    ...
    (uint80 roundID, int256 daiPrice, , uint256 timestamp, uint80 answeredInRound) = DAI.latestRoundData();
    require(daiPrice > 0, "ChainLink: DAI price <= 0");
    require(answeredInRound >= roundID, "ChainLink: Stale price");
    require(timestamp > 0, "ChainLink: Round not complete");

    (roundID, int256 usdcPrice, , timestamp, answeredInRound) = USDC.latestRoundData();
    require(usdcPrice > 0, "ChainLink: USDC price <= 0");
    require(answeredInRound >= roundID, "ChainLink: Stale USDC price");
    require(timestamp > 0, "ChainLink: USDC round not complete");

    (roundID, int256 usdtPrice, , timestamp, answeredInRound) = USDT.latestRoundData();
    require(usdtPrice > 0, "ChainLink: USDT price <= 0");
    require(answeredInRound >= roundID, "ChainLink: Stale USDT price");
    require(timestamp > 0, "ChainLink: USDT round not complete");

    ...
}
```

