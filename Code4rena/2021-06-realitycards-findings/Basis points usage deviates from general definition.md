## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Basis points usage deviates from general definition](https://github.com/code-423n4/2021-06-realitycards-findings/issues/72) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The general definition of basis points is 100 bps = 1%. The usage here, 1000 bps = 100%, deviates from generally accepted definition and could cause confusion among users/creators/affiliates or potential miscalculations.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L228

https://www.investopedia.com/terms/b/basispoint.asp

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Document the used definition of basis points or switch to the generally accepted definition.

