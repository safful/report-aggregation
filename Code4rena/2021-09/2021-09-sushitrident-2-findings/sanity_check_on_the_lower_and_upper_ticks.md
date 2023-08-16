## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Sanity check on the lower and upper ticks](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/93) 

# Handle

broccoli


# Vulnerability details

## Impact

In the `burn` and `swap` functions of `ConcentratedLiquidityPool`, the lower tick is not explicitly checked to be less than the upper tick. Besides, the ticks are not checked to be at least the minimum tick and at most the maximum tick.

## Proof of Concept

Referenced code:
[Ticks.sol#L68-L70](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/libraries/concentratedPool/Ticks.sol#L68-L70)

## Recommended Mitigation Steps

Add sanity checks on the lower and upper ticks in critical functions (see the referenced line of code, for example).

