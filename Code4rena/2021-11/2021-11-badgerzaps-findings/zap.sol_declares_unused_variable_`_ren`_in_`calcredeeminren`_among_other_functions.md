## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Zap.sol declares unused variable `_ren` in `calcRedeemInRen` among other functions](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/4) 

# Handle

TomFrench


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

The variable `_ren` in `Zap.calcRedeemInRen` is declared but unused. This increases gas costs for no benefit.

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L272-L280

This also happens in other functions.

## Recommended Mitigation Steps

Remove unused variable

