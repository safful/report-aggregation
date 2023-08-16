## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Potential division by zero ](https://github.com/code-423n4/2021-06-tracer-findings/issues/83) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In function minimumMargin(), maximumLeverage being zero is not handled because it will result in div by zero as PRBMathUD60x18.div expects non-zero divisor. 

Impact: Various critical market functions will revert if maximumLeverage is zero.

## Proof of Concept

https://github.com/hifi-finance/prb-math/blob/c4dea7d0e6ae246fbb631f7fb4be4072d1da9a07/contracts/PRBMathUD60x18.sol#L71-L77


https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibBalances.sol#L118

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibBalances.sol#L135

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L186

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L242

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L248

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add checks to make sure maximumLeverage is never zero or handle appropriately.

