## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [missing zero-address check ](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/5) 

# Handle

jah


# Vulnerability details

## Impact
The parameter that are used in initialize() function to initialize the state variable,these state variable are used in other function to perform operation. since it lacks zero address validation, it will be problematic if there is error in these state variable. some of the function will loss their functionality which can cause the redeployment of contract

## Proof of Concept
https://github.com/code-423n4/2021-10-badgerdao/blob/9c0ea7b3b02675211446f6c81750c5f3c0a86370/contracts/WrappedIbbtcEth.sol#L37

## Tools Used
Manual Analysis
## Recommended Mitigation Steps
add require condition which check zero address validation


