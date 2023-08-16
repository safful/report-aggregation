## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ConcentratedLiquidityPool: rangeFeeGrowth and secondsPerLiquidity math needs to be unchecked](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/13) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The fee growth mechanism, and by extension, secondsPerLiquidity mechanism of Uniswap V3 has the ability to underflow. It is therefore a necessity for the math to (ironically) be unsafe / unchecked. 

### Proof of Concept

Assume the following scenario and initial conditions:

- Price at parity (nearestTick is 0)
- tickSpacing of 10
- Swaps only increase the price (nearestTick moves up only)
- feeGrowthGlobal initializes with 0, increases by 1 for every tick moved for simplicity
- Existing positions that provide enough liquidity and enable nearestTick to be set to values in the example
- Every tick initialized in the example is ≤ nearestTick, so that its feeGrowthOutside = feeGrowthGlobal

1. When nearestTick is at 40, Alice creates a position for uninitialised ticks [-20, 30]. The ticks are initialized, resulting in their feeGrowthOutside values to be set to 40.
2. nearestTick moves to 50. Bob creates a position with ticks [20, 30] (tick 20 is uninitialised, 30 was initialized from Alice's mint). tick 20 will therefore have a feeGrowthOutside of 50.
3. Let us calculate `rangeFeeGrowth(20,30)`.
    - lowerTick = 20, upperTick = 30
    - feeGrowthBelow = 50 (lowerTick's feeGrowthOutside) since lowerTick < currentTick
    - feeGrowthAbove = 50 - 40 = 10 (feeGrowthGlobal - upperTick's feeGrowthOutside) since upperTick < currentTick
    - feeGrowthInside
        
        = feeGrowthGlobal - feeGrowthBelow - feeGrowthAbove
        
        = 50 - 50 - 10
        
        = -10
        

We therefore have negative feeGrowthInside.

This behaviour is actually acceptable, because the important thing about this mechanism is the relative values to each other, not the absolute values themselves.

### Recommended Mitigation Steps

`rangeFeeGrowth()` and `rangeSecondsInside()` has to be unchecked. In addition, the subtraction of feeGrowthInside values should also be unchecked in `_updatePosition()` and `ConcentratedLiquidityPosition#collect()`.

The same also applies for the subtraction of `pool.rangeSecondsInside` and `stake.secondsInsideLast` in `claimReward()` and `getReward()` of the `ConcentratedLiquidityPoolManager` contract.

