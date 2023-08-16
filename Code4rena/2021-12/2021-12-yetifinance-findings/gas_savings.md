## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/139) 

# Handle

csanuragjain


# Vulnerability details

## Impact
Gas wastage

## Proof of Concept
1. Navigate to contract at https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/YetiCustomBase.sol

2. Observe that in _subColls function the last for loop is not required if n=0 since this means that all token amount is 0


