## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [uint is always >= 0](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/102) 

# Handle

Jujic


# Vulnerability details

## Impact
The uint   `percentBacked ` can not  be negative.

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/PriceCurves/ThreePieceWiseLinearPriceCurve.sol#L131
```
uint256 percentBacked = _collateralVCBalance.mul(1e18).div(_totalVCBalance);
require(percentBacked <= 1e18 && percentBacked >= 0, "percent backed out of bounds");
```

## Tools Used
Remix

## Recommended Mitigation Steps
You should remove the `percentBacked >= 0` from require to save some gas.

