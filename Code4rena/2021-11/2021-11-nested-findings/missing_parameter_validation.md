## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing parameter validation](https://github.com/code-423n4/2021-11-nested-findings/issues/178) 

# Handle

cmichel


# Vulnerability details

Some parameters of functions are not checked for non-zero values:
- `NestedFactory.constructor`: address parameters could be zero or not a contract
- `NestedReserve.constructor`: address parameters could be zero or not a contract
- `NestedBuybacker.constructor`: address parameters could be zero or not a contract

## Impact
Wrong user input or wallets defaulting to the zero addresses for a missing input can lead to the contract needing to redeploy or wasted gas.

## Recommended Mitigation Steps
Validate the parameters.


