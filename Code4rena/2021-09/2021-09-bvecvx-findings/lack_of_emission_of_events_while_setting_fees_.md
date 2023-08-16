## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [lack of emission of events while setting fees ](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/60) 

# Handle

JMukesh


# Vulnerability details

## Impact
setWithdrawalFee(), setPerformanceFeeStrategist()  has no event, so it is difficult to track off-chain changes in the fee 

## Proof of Concept
https://github.com/code-423n4/2021-09-bvecvx/blob/1d64bd58c7a4224cc330cef283561e90ae6a3cf5/veCVX/deps/BaseStrategy.sol#L126

## Tools Used
manual review

## Recommended Mitigation Steps
add event to above function

