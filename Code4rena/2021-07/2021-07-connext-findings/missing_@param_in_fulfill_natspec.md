## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing @param in fulfill NatSpec](https://github.com/code-423n4/2021-07-connext-findings/issues/64) 

# Handle

0xsanson


# Vulnerability details

## Impact
The currect implementation of NatSpec of fulfill function lacks @param callData

## Proof of Concept
https://github.com/code-423n4/2021-07-connext/blob/main/contracts/TransactionManager.sol#L302

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
It's suggested to complete adding @param callData

