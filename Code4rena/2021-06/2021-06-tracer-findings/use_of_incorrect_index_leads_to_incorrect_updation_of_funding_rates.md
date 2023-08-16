## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Use of incorrect index leads to incorrect updation of funding rates](https://github.com/code-423n4/2021-06-tracer-findings/issues/74) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The updateFundingRate() function updates the funding rate and insurance funding rate. While the instant/new funding rates are calculated correctly, the cumulative funding rate calculation is incorrect because it is always adding the instant to 0, not the previous value. This is due to the use of [currentFundingIndex] which has been updated since the previous call to this function while it should really be using [currentFundingIndex-1] to reference the previous funding rate.

Impact: The cumulative funding rate and insurance funding rates are calculated incorrectly without considering the correct previous values. This affects the settling of accounts across the entire protocol. The protocol logic is significantly impacted, accounts will not be settled as expected, protocol shutdown and contracts will need to be redeployed. Users may lose funds and protocol takes a reputation hit.
 
## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L155-L160

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L168

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L77

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L196-L215

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Pricing.sol#L221-L230

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L445-L446

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use [currentFundingIndex-1] for non-zero values of currentFundingIndex to get the value updated in the previous call on lines L155 and L159 of Pricing.sol.

