## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [tvl calculation in withdraw() should use convertedWadAmount instead of amount](https://github.com/code-423n4/2021-06-tracer-findings/issues/57) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The tvl calculation in deposit() uses convertedWadAmount but the one in withdraw() uses the parameter amount. While amount is still in WAD format, it may contain dust which is what the conversion to rawTokenAmount and then back to convertedWadAmount removes.

Impact: Use of amount in tvl during withdraw() will consider dust while the one in deposit() will not, which is inconsistent.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L200

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L176-L177

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L162

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L153-L155

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use convertedWadAmount instead of amount to be consistent with the increment during withdraw() tvl calculation.

