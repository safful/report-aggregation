## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ConcentratedLiquidityPool: initialPrice should be checked to be within allowable range](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/11) 

# Handle

hickuphh3


# Vulnerability details

### Impact

No check is performed for the initial price. This means that it can be set to be below the `MIN_SQRT_RATIO` or above `MAX_SQRT_RATIO` (Eg. zero value), which will prevent the usability of all other functions (minting, swapping, burning).

For example, `Ticks.insert()` would fail when attempting to calculate `actualNearestTick = TickMath.getTickAtSqrtRatio(currentPrice);`, which means no one will be able to mint positions.

### Recommended Mitigation Steps

Check the `initialPrice` is within the acceptable range, ie. `MIN_SQRT_RATIO <= initialPrice <= MAX_SQRT_RATIO`

