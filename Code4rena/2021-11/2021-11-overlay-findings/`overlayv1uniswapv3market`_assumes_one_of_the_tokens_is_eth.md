## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`OverlayV1UniswapV3Market` assumes one of the tokens is ETH](https://github.com/code-423n4/2021-11-overlay-findings/issues/79) 

# Handle

cmichel


# Vulnerability details

The `OverlayV1UniswapV3Market` contract assumes that one of the tokens of `_ovlFeed` is ETH but does not check it in the constructor:

```solidity
constructor(
    address _mothership,
    address _ovlFeed,
    address _marketFeed,
    address _quote,
    address _eth,
    uint128 _baseAmount,
    uint256 _macroWindow,
    uint256 _microWindow,
    uint256 _priceFrameCap
) OverlayV1Market (
    _mothership
) OverlayV1Comptroller (
    _microWindow
) OverlayV1OI (
    _microWindow
) OverlayV1PricePoint (
    _priceFrameCap
) {

    // immutables
    eth = _eth;
    // could be that token1 is not ETH either
    ethIs0 = IUniswapV3Pool(_ovlFeed).token0() == _eth;
    // ...
}
```

## Impact
If `token0` is _not_ ETH, then it assumes `token1` is ETH but never validates this assumption.
This could lead to wrong market liquidity and prices calculations if an `_ovlFeed` is supplied that is not actually the OVL/ETH feed.

## Recommended Mitigation Steps
Check that `(token0 == OVL && token1 == WETH) || (token1 == OVL && token0 == WETH)` for `_ovlFeed`.

