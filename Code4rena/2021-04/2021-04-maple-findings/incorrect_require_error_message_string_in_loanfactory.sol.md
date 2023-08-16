## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Incorrect require error message string in LoanFactory.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/65) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The error message string for the require statement on L167 of LoanFactory.sol incorrectly uses PoolFactory as the source contract for this message instead of LoanFactory, which could be confusing when this error is hit.

 require(!globals.protocolPaused(), "PoolFactory:PROTOCOL_PAUSED");

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/LoanFactory.sol#L167

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change error message to:
require(!globals.protocolPaused(), “LoanFactory:PROTOCOL_PAUSED");


