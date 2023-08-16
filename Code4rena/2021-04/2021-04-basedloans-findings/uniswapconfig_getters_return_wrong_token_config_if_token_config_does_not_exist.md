## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [UniswapConfig getters return wrong token config if token config does not exist](https://github.com/code-423n4/2021-04-basedloans-findings/issues/37) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `UniswapConfig.getTokenConfigBySymbolHash` function does not work as `getSymbolHashIndex` returns `0` if there is no config token for that symbol (uninitialized map value), but the outer function implements the non-existence check with `-1`.

The same issue occurs also for:

- `getTokenConfigByCToken`
- `getTokenConfigByUnderlying`

## Impact

When encountering a non-existent token config, it will always return the token config of the **first index** (index 0) which is a valid token config for a completely different token.
This leads to wrong oracle prices for the actual token which could in the worst case be used to borrow more tokens at a lower price or borrow more tokens by having a higher collateral value, essentially allowing undercollateralized loans that cannot be liquidated.

## Recommended Mitigation Steps

Fix the non-existence check.


