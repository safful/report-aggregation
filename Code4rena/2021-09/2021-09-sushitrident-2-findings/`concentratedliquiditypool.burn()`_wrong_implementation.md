## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`ConcentratedLiquidityPool.burn()` Wrong implementation](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/24) 

# Handle

WatchPug


# Vulnerability details

The reserves should be updated once LP tokens are burned to match the actual total bento shares hold by the pool.

However, the current implementation only updated reserves with the fees subtracted.

Makes the `reserve0` and `reserve1` smaller than the current `balance0` and `balance1`.

## Impact

As a result, many essential features of the contract will malfunction, includes `swap()` and `mint()`.

## Recommended Mitigation Steps

https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPool.sol#L263-L267

Change:

```
        unchecked {
            reserve0 -= uint128(amount0fees);
            reserve1 -= uint128(amount1fees);
        }

```
to:

```
        unchecked {
            reserve0 -= uint128(amount0);
            reserve1 -= uint128(amount1);
        }
```

