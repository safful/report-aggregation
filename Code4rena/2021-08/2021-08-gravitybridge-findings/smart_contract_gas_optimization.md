## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Smart Contract Gas Optimization](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/3) 

# Handle

ElliotFriedman


# Vulnerability details

## Impact
Currently, submitBatch, updateValset, deployERC20 and submitLogicCall all have arguments that are in memory. This causes calls to these functions to be more expensive than they need to be. By moving to external functions and upgrading the compiler version, there will be gas savings. The larger the amount of data being submitted to the contracts, the greater the savings as the cost of memory in the EVM goes up quadratically with the amount of data stored.

## Tools Used
Hardhat

## Recommended Mitigation Steps
Make all functions that you can external instead of public, especially the ones mentioned above that will see large transaction volumes, and change the data types from memory to external. This may involve changing compiler versions to 0.8.0 or greater to support using structs as external types.

