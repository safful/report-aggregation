## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Differing percentage denominators causes confusion and potentially brick claims](https://github.com/code-423n4/2022-03-joyn-findings/issues/53) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L14
https://github.com/code-423n4/2022-03-joyn/blob/main/splits/contracts/Splitter.sol#L103


# Vulnerability details

## Details & Impact

There is a `PERCENTAGE_SCALE = 10e5` defined, but the actual denominator used is `10000`. This is aggravated by the following factors:

1. Split contracts are created by collection owners, not the factory owner. Hence, there is a likelihood for someone to mistakenly use `PERCENTAGE_SCALE` instead of `10000`.
2. The merkle root for split distribution can only be set once, and a collection’s split and royalty vault can’t be changed once created.

Thus, if an incorrect denominator is used, the calculated claimable amount could exceed the actual available funds in the contract, causing claims to fail and funds to be permanently locked.

## Recommended Mitigation Steps

Remove `PERCENTAGE_SCALE` because it is unused, or replace its value with `10_000` and use that instead. 

P.S: there is an issue with the example scaled percentage given for platform fees `(5% = 200)`. Should be `500` instead of `200`.

