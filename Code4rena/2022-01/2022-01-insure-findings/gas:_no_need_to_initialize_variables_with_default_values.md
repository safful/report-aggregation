## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: No need to initialize variables with default values](https://github.com/code-423n4/2022-01-insure-findings/issues/40) 

# Handle

Dravee


# Vulnerability details

## Impact  
If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.
  
## Proof of Concept  
Instances include:  
```  
Factory.sol:176:            for (uint256 i = 0; i < _references.length; i++) {
Factory.sol:186:            for (uint256 i = 0; i < _conditions.length; i++) {
IndexTemplate.sol:280:            for (uint256 i = 0; i < _length; i++) {
IndexTemplate.sol:348:        for (uint256 i = 0; i < _length; i++) {
IndexTemplate.sol:381:        for (uint256 i = 0; i < _length; i++) {
IndexTemplate.sol:462:        for (uint256 i = 0; i < _poolLength; i++) {
IndexTemplate.sol:655:        for (uint256 i = 0; i < poolList.length; i++) {
PoolTemplate.sol:343:        for (uint256 i = 0; i < _ids.length; i++) {
PoolTemplate.sol:671:        for (uint256 i = 0; i < indexList.length; i++) {
PoolTemplate.sol:703:        for (uint256 i = 0; i < indexList.length; i++) {
Vault.sol:109:        for (uint128 i = 0; i < 2; i++) {
```  
  
## Tools Used  
Manual Analysis  
  
## Recommended Mitigation Steps  
Remove explicit initialization for default values.


