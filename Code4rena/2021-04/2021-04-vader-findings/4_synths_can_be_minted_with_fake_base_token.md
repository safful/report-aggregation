## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- filed

# [4 Synths can be minted with fake base token](https://github.com/code-423n4/2021-04-vader-findings/issues/207) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `Pools.mintSynth` function does not check if `base` is one of the base tokens. One can transfer `token`s to the pool and set `base=token` and call `mintSynth(token, token, member)`.

The `_actualInput = getAddedAmount(base, token);` will return the **token** amount added but use the ratio compared to the **base** reserve `calcSwapOutput(_actualInput=tokenInput, mapToken_baseAmount[token], mapToken_tokenAmount[token]); = tokenIn / baseAmount * tokenAmount` which yields a wrong swap result.

## Impact

It breaks the accounting for the pool as `token`s are transferred in, but the `base` balance is increased.

The amount that is minted could also be inflated (cheaper than sending the actual base tokens), especially if `token` is a high-precision token or worth less than base.

## Recommended Mitigation Steps

Check that `base` is either `USDV` or `VADER` in `mintSynth`.


