## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Inconsistent NatSpec comment in DebtLocker.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/34) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Access control of external/public functions via modifiers or require statements is typically specified in the @dev part of the NatSpec comment. This highlight is missing for the claim() function which is accessible only by isPool.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/DebtLocker.sol#L44-L54

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add “Only called by the pool contract.” to @dev on L45.


