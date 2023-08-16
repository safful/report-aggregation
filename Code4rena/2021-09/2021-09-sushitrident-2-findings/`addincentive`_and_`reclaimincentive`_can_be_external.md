## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`addIncentive` and `reclaimIncentive` can be external](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/75) 

# Handle

0xsanson


# Vulnerability details

## Impact
Function `addIncentive` and `reclaimIncentive` in ConcentratedLiquidityPoolManager can be `external` instead of `public` to save gas.

## Proof of Concept
https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPoolManager.sol#L36
https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPoolManager.sol#L49

## Tools Used
editor

