## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [coolDown.redeemWindowEnd serves no purpose](https://github.com/code-423n4/2022-01-notional-findings/issues/43) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
Increases gas costs due to manipulating redeemWindowEnd and usage of structs for `AccountCooldown`

## Proof of Concept

`redeemWindowEnd` will always equal `redeemWindowBegin + REDEEM_WINDOW_SECONDS` so it can just be calculated when needed (excluding the situation where both of these are zero which is already handled in the code.)

We then don't need to store both of these values in storage and deal with the overhead of using structs.

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L244

We can then just replace this line with 

```
block.timestamp < coolDown.redeemWindowBegin + REDEEM_WINDOW_SECONDS
```

## Recommended Mitigation Steps

Remove `coolDown.redeemWindowEnd` and store cooldowns as a simple uint rather than a struct.

