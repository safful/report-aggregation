## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Usage of a non-native 256 bits uint as a counter in for-loops increases gas cost](https://github.com/code-423n4/2022-01-insure-findings/issues/62) 

# Handle

Dravee


# Vulnerability details

## Impact  
Due to how the EVM natively works on 256 bit numbers, using a 8 bit number in for-loops introduces additional costs as the EVM has to properly enforce the limits of this smaller type.

See the warning at this link: https://docs.soliditylang.org/en/v0.8.0/internals/layout_in_storage.html#layout-of-state-variables-in-storage :
> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
> It is only beneficial to use reduced-size arguments if you are dealing with storage values because the compiler will pack multiple elements into one storage slot, and thus, combine multiple reads or writes into a single operation. When dealing with function arguments or memory values, there is no inherent benefit because the compiler does not pack these values.
  
## Proof of Concept  
```
Vault.sol:109:        for (uint128 i = 0; i < 2; i++) {
``` 
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Use `uint256` as a counter in for-loops.


