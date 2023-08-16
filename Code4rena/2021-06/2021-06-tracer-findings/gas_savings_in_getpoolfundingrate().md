## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings in getPoolFundingRate()](https://github.com/code-423n4/2021-06-tracer-findings/issues/125) 

# Handle

0xsanson


# Vulnerability details

## Impact
We can save gas by substituting getPoolTarget() with levNotionalValue/100, since tracer.leveragedNotionalValue() is already saved in memory.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L216

## Tools Used
Manual analysis.

## Recommended Mitigation Steps
Substitute getPoolTarget() with levNotionalValue/100.

