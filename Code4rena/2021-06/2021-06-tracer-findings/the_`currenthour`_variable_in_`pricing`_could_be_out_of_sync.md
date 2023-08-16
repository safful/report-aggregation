## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [The `currentHour` variable in `Pricing` could be out of sync](https://github.com/code-423n4/2021-06-tracer-findings/issues/142) 

# Handle

shw


# Vulnerability details

## Impact

The `recordTrade` function in `Pricing` updates the `currentHour` variable by 1 every hour. However, if there is no trade (i.e., the `recordTrade` is not called) during this hour, the `currentHour` is out of sync with the actual hour. As a result, the `averagePriceForPeriod` function uses the prices before 24 hours and causes errors on the average price.

## Proof of Concept

Referenced code:
[Pricing.sol#L90-L94](https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L90-L94)

## Recommended Mitigation Steps

Calculate how much time passed (e.g., `(block.timestamp - startLastHour) / 3600`) to update the `currentHour` variable correctly.

