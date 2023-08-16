## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`Prices.averagePrice` does not show a difference between no trades and a zero price](https://github.com/code-423n4/2021-06-tracer-findings/issues/139) 

# Handle

shw


# Vulnerability details

## Impact

The `getHourlyAvgTracerPrice` and `getHourlyAvgOraclePrice` functions in `Pricing` return 0 if there is no trade during the given `hour` because of the design of `averagePrice`, which could mislead users that the hourly average price is 0. The same problem happens when emitting the old hourly average in the `recordTrade` function.

## Proof of Concept

Referenced code:
[Pricing.sol#L254-L256](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L254-L256)
[Pricing.sol#L262-L264](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L262-L264)
[Pricing.sol#L74](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L74)

## Recommended Mitigation Steps

Return a special value (e.g., `type(uint256).max`) from `averagePrice` if there is no trade during the specified hour to distinguish from an actual zero price. Handle this particular value whenever the `averagePrice` function is called by others.

