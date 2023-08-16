## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`ConcentratedLiquidityPosition.sol#collect()` Users may get double the amount of yield when they call `collect()` before `burn()`](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/53) 

# Handle

WatchPug


# Vulnerability details

When a user calls `ConcentratedLiquidityPosition.sol#collect()` to collect their yield, it calcuates the yield based on `position.pool.rangeFeeGrowth()` and `position.feeGrowthInside0, position.feeGrowthInside1`:

https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPosition.sol#L75-L101

When there are enough tokens in `bento.balanceOf`, it will not call `position.pool.collect()` to collect fees from the pool.

This makes the user who `collect()` their yield when there is enough balance to get double yield when they call `burn()` to remove liquidity. Because `burn()` will automatically collect fees on the pool contract.

## Impact

The yield belongs to other users will be diluted.

## Recommended Mitigation Steps

Consider making `ConcentratedLiquidityPosition.sol#burn()` call `position.pool.collect()` before `position.pool.burn()`. User will need to call `ConcentratedLiquidityPosition.sol#collect()` to collect unclaimed fees after `burn()`.

Or `ConcentratedLiquidityPosition.sol#collect()` can be changed into a `public` method and `ConcentratedLiquidityPosition.sol#burn()` can call it after `position.pool.burn()`.

