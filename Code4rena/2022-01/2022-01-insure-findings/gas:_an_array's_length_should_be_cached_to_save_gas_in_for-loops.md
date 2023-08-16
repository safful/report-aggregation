## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: An array's length should be cached to save gas in for-loops](https://github.com/code-423n4/2022-01-insure-findings/issues/43) 

# Handle

Dravee


# Vulnerability details

## Impact  
Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.  
  
Caching the array length in the stack saves around 3 gas per iteration.  
  
## Proof of Concept  
```  
Factory.sol:176:            for (uint256 i = 0; i < _references.length; i++) {
Factory.sol:186:            for (uint256 i = 0; i < _conditions.length; i++) {
IndexTemplate.sol:655:        for (uint256 i = 0; i < poolList.length; i++) {
PoolTemplate.sol:343:        for (uint256 i = 0; i < _ids.length; i++) {
PoolTemplate.sol:671:        for (uint256 i = 0; i < indexList.length; i++) {
PoolTemplate.sol:703:        for (uint256 i = 0; i < indexList.length; i++) {
```  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Store the array's length in a variable before the for-loop, and use it instead.


