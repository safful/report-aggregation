## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Conversions between sNOTE and BPT when burning cause less sNOTE to be burned than expected](https://github.com/code-423n4/2022-01-notional-findings/issues/71) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

`sNOTE.redeem` burns an amount of sNOTE other than `sNOTEAmount`, potentially giving a better rate to redeemers than it should.

## Proof of Concept

Following the process of burning sNOTE for BPT:

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L238-L252

1. We pass an amount of sNOTE to burn which is converted into BPT. (L248)

https://github.com/code-423n4/2022-01-notional/blob/d171cad9e86e0d02e0909eb66d4c24ab6ea6b982/contracts/sNOTE.sol#L315-L323

2. We calculate the what fraction of the total amount of BPT this user is entitled to this amount of BPT represents and then multiply that by their balance of sNOTE (L320)

Rather than burning `sNOTEAmount` we then end up burning 

```
realSNOTEAmount = balanceOf(account) * getPoolTokenShare(sNOTEAmount) / getPoolTokenShare(balanceOf(account))
```

This looks to round down such that we end up burning less sNOTE than expected. We're then going to be giving the user a slightly better rate of BPT for sNOTE than we should whereas any rounding should be in favour of the sNOTE contract.

In any case, we're performing many reads from storage (including from other contracts) in order to calculate a value which was originally passed by the user so using `sNOTEAmount` directly would be a gas optimisation.

## Recommended Mitigation Steps

Change `_burn` to take an amount of sNOTE to burn as an argument rather than an amount of BPT which is equivalent to the amount of sNOTE to be burnt. `sNOTEAmount` can then be passed from the `redeem` function directly. `bptToRedeem` will still be rounded down so sNOTE will always have favourable rounding.

