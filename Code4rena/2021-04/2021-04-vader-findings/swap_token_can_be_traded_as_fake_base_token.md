## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- filed

# [Swap token can be traded as fake base token](https://github.com/code-423n4/2021-04-vader-findings/issues/205) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `Pools.swap` function does not check if `base` is one of the base tokens. One can transfer `token`s to the pool and set `base=token` and call `swap(token, token, member, toBase=false)`.

The `_actualInput = getAddedAmount(base, token);` will return the **token** amount added but use the ratio compared to the **base** reserve `calcSwapOutput(_actualInput=tokenInput, mapToken_baseAmount[token], mapToken_tokenAmount[token]); = tokenIn / baseAmount * tokenAmount` which yields a wrong swap result.

## Impact

It breaks the accounting for the pool as `token`s are transferred in, but the `base` balance is increased (and `token` decreased). LPs cannot correctly withdraw again, and others cannot correctly swap again.

Another example scenario is that the token pool amount can be stolen.
Send `tokenIn=baseAmount` of tokens to the pool and call `swap(base=token, token, member, toBase=false)`. Depending on the price of `token` relative to `base` this could be cheaper than trading with the base tokens.

## Recommended Mitigation Steps

Check that `base` is either `USDV` or `VADER`

