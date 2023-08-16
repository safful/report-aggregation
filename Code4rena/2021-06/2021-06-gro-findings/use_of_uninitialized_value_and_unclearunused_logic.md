## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Use of uninitialized value and unclear/unused logic](https://github.com/code-423n4/2021-06-gro-findings/issues/65) 

# Handle

0xRajeev


# Vulnerability details

## Impact

vaultIndexes is uninitialized and it's unclear what 10000 signifies here. investDelta return value is also ignored at call site. If this is an indication of missed/incorrect logic, then it's risky. If not, removing will help readability/maintainability.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L166

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L155

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate any missing logic or else remove unused code.

