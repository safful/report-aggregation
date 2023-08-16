## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Rearranging order of state variable declarations to pack them will save storage slots and gas](https://github.com/code-423n4/2021-06-gro-findings/issues/33) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Moving declarations of state variables that take < 32 Bytes next to each other will allow combining them in the same storage slot and potentially save gas from combined SSTOREs depending on store patterns.

Impact: Moving emergencyState bool right next to preventSmartContracts bool will conserve a storage slot and may save gas.

## Proof of Concept

See reference: https://mudit.blog/solidity-gas-optimization-tips/ and https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L54

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L44


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Moving declarations of state variables that take < 32 Bytes next to each other. E.g.: booleans next to each other or address types.

