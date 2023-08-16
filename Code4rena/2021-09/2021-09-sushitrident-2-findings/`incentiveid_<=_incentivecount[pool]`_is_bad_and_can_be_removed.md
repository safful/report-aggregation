## Tags

- bug
- duplicate
- 0 (Non-critical)
- sponsor confirmed

# [`incentiveId <= incentiveCount[pool]` is bad and can be removed](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/79) 

# Handle

0xsanson


# Vulnerability details

## Impact
When an user subscribes to an incentive using ConcentratedLiquidityPoolManager's `subscribe`, the function checks that `incentiveId` is appropriate:
```js
require(incentiveId <= incentiveCount[pool], "NOT_INCENTIVE");
```
This check is actually incorrect, and it should use a `<` instead of `<=`.

If this was the only requirement, it would be possible to subscribe to the next incentive, causing some problems. Fortunately the next line saves the day:
`require(block.timestamp > incentive.startTime && block.timestamp < incentive.endTime, "TIMED_OUT");` this fails for uninitiated incentives.

## Proof of Concept
https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPoolManager.sol#L72

## Tools Used
editor

## Recommended Mitigation Steps
Consider removing this requirement to save gas. The check for existing pool is already considered when looking at `block.timestamp < incentive.endTime`.

