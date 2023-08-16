## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Protocol uses floating pragmas ](https://github.com/code-423n4/2022-01-livepeer-findings/issues/47) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In files like L1LPTDataCache.sol,  floating pragmas are used.  Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

## Proof of Concept
https://swcregistry.io/docs/SWC-103

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Lock the pragma version:  delete pragma solidity 0.8.0 in favor of pragma solidity 0.8.0

