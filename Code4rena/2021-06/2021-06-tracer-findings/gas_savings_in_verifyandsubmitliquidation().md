## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings in verifyAndSubmitLiquidation()](https://github.com/code-423n4/2021-06-tracer-findings/issues/131) 

# Handle

0xsanson


# Vulnerability details

## Impact
In Liquidation.verifyAndSubmitLiquidation(...) we can save the minimumMargin to memory since it's called two times.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Liquidation.sol#L171

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Save the result of Balances.minimumMargin(...) to memory.

