## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [UniswapV2/SushiwapLPAdapter update the wrong token](https://github.com/code-423n4/2021-10-mochi-findings/issues/134) 

# Handle

cmichel


# Vulnerability details

The `UniswapV2LPAdapter/SushiswapV2LPAdapter.update` function retrieves the `underlying` from the LP token pair (`_asset`) but then calls `router.update(_asset, _proof)` which is the LP token itself again.
This will end up with the router calling this function again recursively.

## Impact
This function fails as there's an infinite recursion and eventually runs out of gas.

## Recommendation
The idea was most likely to update the `underlying` price which is used in `_getPrice` as `uint256 eAvg = cssr.getExchangeRatio(_underlying, weth);`.

Call `router.update(underlying, _proof)` instead. Note that the `_proof` does not necessarily update the `underlying <> WETH` pair, it could be any `underlying <> keyAsset` pair.


