## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing checks for lowestMaxLeverage < maxLeverage and insurancePoolSwitchStage < deleveragingCliff](https://github.com/code-423n4/2021-06-tracer-findings/issues/117) 

# Handle

0xsanson


# Vulnerability details

## Impact
The contract TracerPerpetualSwaps introduces these four state variables (lowestMaxLeverage,  maxLeverage, insurancePoolSwitchStage and deleveragingCliff) and four respective set functions. Logically the following relations are needed: lowestMaxLeverage < maxLeverage and insurancePoolSwitchStage < deleveragingCliff, but the code doesn't check for them.

Non-critical because needs an error by the owner.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L552
Also lines L560, L564, L568

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Add appropriate requires to the set functions and the constructor.

