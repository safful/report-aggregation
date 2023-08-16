## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Event log poisoning/griefing in withdrawFees() ](https://github.com/code-423n4/2021-06-tracer-findings/issues/63) 

# Handle

0xRajeev


# Vulnerability details

## Impact

withdrawFees() is an external function which can be called by anyone to transfer the accumulated fees to the feeReceiver account. However, there is no data validation to check if fees are non-zero.

Impact: One can keep calling withdrawFees(), even if the fees is zero, to grief the system with 0 amount transfers and emission of events recording the same. This leads to what is known as event log poisoning where malicious external users spam the Tracer contract to generate arbitrary FeeWithdrawn events.

## Proof of Concept

See similar Finding from Sigma Prime’s audit of Synthetix Unipool: https://github.com/sigp/public-audits/blob/master/synthetix/unipool/review.pdf

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L508-L516

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider adding a require or if statement preventing the withdrawFees() function from emitting the event when the amount variable is zero, i.e. check if fees != 0 before transfer+emit.

