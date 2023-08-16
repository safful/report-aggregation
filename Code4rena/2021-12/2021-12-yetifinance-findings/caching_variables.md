## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching variables](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/70) 

# Handle

Jujic


# Vulnerability details

## Impact
Some of the variable can be cached to slightly reduce gas usage

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/PriceCurves/ThreePieceWiseLinearPriceCurve.sol#L90-L91

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/PriceCurves/ThreePieceWiseLinearPriceCurve.sol#L107-L108

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/PriceCurves/ThreePieceWiseLinearPriceCurve.sol#L144-L145

```
dollarCap // can be cashed
decayTime // can be cashed

```
## Tools Used
Remix

## Recommended Mitigation Steps


