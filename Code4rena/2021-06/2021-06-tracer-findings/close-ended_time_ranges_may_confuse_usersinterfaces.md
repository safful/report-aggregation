## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Close-ended time ranges may confuse users/interfaces](https://github.com/code-423n4/2021-06-tracer-findings/issues/75) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Time ranges are typically open-ended which includes the start & end times, and not close-ended. So the releaseTime would be interpreted as the time it would be released i.e. block.timestamp >= releaseTime would be the expected check here instead of ‘>’. Similarly, on L406, it should be ‘<=‘ instead of ‘<‘.

Impact: Claims of escrow and receipts are expected to succeed in a particular block but they revert and have to wait until the next block.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L112

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L406

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Unless justified, change the strict inequality and make it ‘>=‘ and ‘<=‘ to convert open ranges to closed ranges for block.timestamp comparisons.

