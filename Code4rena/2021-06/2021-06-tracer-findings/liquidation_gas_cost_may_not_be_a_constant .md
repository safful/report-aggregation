## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [LIQUIDATION_GAS_COST may not be a constant ](https://github.com/code-423n4/2021-06-tracer-findings/issues/48) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The gas cost for liquidation may change if code is updated/optimized, compiler changed or profiling improved. The developers may forget to update this constant in code.

Impact: The margin validity calculation which uses this value may be affected if this changes and hence is not as declared in the constant. This may adversely impact validation.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L26

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L244

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L250

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L494

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L159

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L193

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

It is safer to make this a constructor-set immutable value that will force usage of an updated accurate value at deployment time. Evaluate if the sensitivity to this value is great enough to justify a setter to change it if incorrectly initialized at deployment.

