## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`OverlayV1UniswapV3Market` computes wrong market liquidity](https://github.com/code-423n4/2021-11-overlay-findings/issues/83) 

# Handle

cmichel


# Vulnerability details

The `OverlayV1UniswapV3Market.fetchPricePoint` tries to compute the market depth in OVL terms as `marketLiquidity (in ETH) / ovlPrice (in ETH per OVL)`.
To get the market liquidity _in ETH_ (and not the other token pair), it uses the `ethIs0` boolean.

```solidity
_marketLiquidity = ethIs0
    ? ( uint256(_liquidity) << 96 ) / _sqrtPrice
    : FullMath.mulDiv(uint256(_liquidity), _sqrtPrice, X96);
```

However, `ethIs0` boolean refers to the `ovlFeed`, whereas the `_liquidity` refers to the `marketFeed`, and therefore the `ethIs0` boolean has nothing to do with the _market_ feed where the liquidity is taken from:

```solidity
// in constructor, if token0 is eth refers to ovlFeed
ethIs0 = IUniswapV3Pool(_ovlFeed).token0() == _eth;

// in fetchPricePoint, _liquidity comes from different market feed
( _ticks, _liqs ) = IUniswapV3Pool(marketFeed).observe(_secondsAgo);
_marketLiquidity = ethIs0
    ? ( uint256(_liquidity) << 96 ) / _sqrtPrice
    : FullMath.mulDiv(uint256(_liquidity), _sqrtPrice, X96);
```

## Impact
If the `ovlFeed` and `marketFeed` do not have the same token position for the ETH pair (ETH is either token 0 or token 1 for **both** pairs), then the market liquidity & depth is computed wrong (inverted).
For example, the `OverlayV1Market.depth()` function will return a wrong depth which is used in the market cap computation.

## Recommended Mitigation Steps
It seems that `marketFeed.token0() == WETH` should be used in `fetchPricePoint` to compute the liquidity instead of `ovlFeed.token0() == WETH`.


