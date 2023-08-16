## Tags

- bug
- sponsor confirmed
- 2 (Med Risk)

# [Wrong keeper reward computation](https://github.com/code-423n4/2021-10-tracer-findings/issues/23) 

# Handle

cmichel


# Vulnerability details

The `PoolKeeper.keeperReward` computation mixes WADs and Quads which leads to issues.
1. Note that `keeperTip` returns values where `1` = `1%`, and `100 = 100%`, the same way `BASE_TIP = 5 = 5%`. Thus `_tipPercent = ABDKMathQuad.fromUInt(keeperTip)` is a Quad value of this keeper tip, and not in "wad units" as the comment above it says.

```solidity
// @audit 👇 this comment is not correct, it's in Quad units
// tip percent in wad units
bytes16 _tipPercent = ABDKMathQuad.fromUInt(keeperTip(_savedPreviousUpdatedTimestamp, _poolInterval));
```

2. Now the `wadRewardValue` interprets `_tipPercent` as a WAD + Quad value which ultimately leads to significantly fewer keeper rewards:
It tries to compute `_keeperGas + _keeperGas * _tipPercent` and to compute `_keeperGas * _tipPercent` it does a wrong division by `fixedPoint` (1e18 as a quad value) because it think the `_tipPercent` is a WAD value (100%=1e18) as a quad, when indeed `100%=100`. It seems like it should divide by `100` as a quad instead.

```
ABDKMathQuad.add(
    ABDKMathQuad.fromUInt(_keeperGas),
    // @audit there's no need to divide by fixedPoint, he wants _keeperGas * _tipPercent and _tipPercent is a quad quad_99 / quad_100
    ABDKMathQuad.div((ABDKMathQuad.mul(ABDKMathQuad.fromUInt(_keeperGas), _tipPercent)), ABDKMathQuad.fromUInt(100))
)
```

## Impact
The keeper rewards are off as the `_keeperGas * _tipPercent` is divided by 1e18 instead of 1e2.
Keeper will just receive their `_keeperGas` cost but the tip part will be close to zero every time.

## Recommended Mitigation Steps
Generally, I'd say the contract mixes quad and WAD units where it doesn't have to do it. Usually, you either use WAD or Quad math but not both at the same time.
This complicates the code.
I'd make `keeperTip()`  return a `byte16` Quad value as a percentage where `100% = ABDKMathQuad.fromUInt(1)`. This temporary float result can then be used in a different ABDKMathQuad computation.

Alternatively, divide by 100 as a quad instead of 1e18 as a quad because `_tipPercent` is not a WAD value, but simply a percentage where `1 = 1%`.

```solidity
ABDKMathQuad.add(
    ABDKMathQuad.fromUInt(_keeperGas),
    // @audit there's no need to divide by fixedPoint, he wants _keeperGas * _tipPercent and _tipPercent is a quad quad_99 / quad_100
    ABDKMathQuad.div((ABDKMathQuad.mul(ABDKMathQuad.fromUInt(_keeperGas), _tipPercent)), ABDKMathQuad.fromUInt(100))
)
```


