## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Missing length check on array could lead to undefined behavior](https://github.com/code-423n4/2021-06-tracer-findings/issues/79) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The sumN() library function expects to calculate the sum of n elements of the supplied array but there is no check to see if the array indeed has n elements. A smaller array could lead to reading out of bounds memory resulting in undefined values.

Impact: The current usage of the library does not indicate an out of bounds access but any new code using this library could be impacted.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibMath.sol#L38-L46

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibMath.sol#L68

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibPrices.sol#L73


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add require(n <= arr.length) at the beginning of sumN() to be safe.

