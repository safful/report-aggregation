## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- filed

# [`getAddedAmount` can return wrong results](https://github.com/code-423n4/2021-04-vader-findings/issues/206) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `getAddedAmount` function only works correctly when called with `(VADER/USDV, pool)` or `(pool, pool)`.
However, when called with (`token, pool)` where `token` is neither `VADER/USDV/pool`, it returns wrong results:

1. It gets the `token` balance
2. And subtracts it from the stored `mapToken_tokenAmount[_pool]` amount which can be that of a completely different token

## Impact

Anyone can break individual pairs by calling `sync(token1, token2)` where the `token1` balance is less than `mapToken_tokenAmount[token2]`. This will add the difference to `mapToken_tokenAmount[token2]` and break the accounting and result in a wrong swap logic.

Furthermore, this can also be used to swap tokens without having to pay anthing with `swap(token1, token2, member, toBase=false)`.

## Recommended Mitigation Steps

Add a require statement in the `else` branch that checks that `_token == _pool`.


