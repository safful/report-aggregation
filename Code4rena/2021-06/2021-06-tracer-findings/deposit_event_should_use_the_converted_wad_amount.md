## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Deposit event should use the converted WAD amount](https://github.com/code-423n4/2021-06-tracer-findings/issues/56) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The Deposit event uses the function parameter amount instead of the convertedWadAmount which is what is used to update the user’s position and tvl because it prevents any dust deposited in amount. This will also make it consistent with the emit event in withdraw function.

Impact: Deposit event amount reflects the value with dust while the user position does not. This may lead to confusion.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L163

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L153-L162

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L204

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use uint256(convertedWadAmount) instead of amount in Deposit event.

