## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Unused Named Returns](https://github.com/code-423n4/2022-01-insure-findings/issues/63) 

# Handle

Dravee


# Vulnerability details

## Impact  
Using both named returns and a return statement isn't necessary.
Removing unused named return variables can reduce gas usage and improve code clarity. To save gas and improve code quality: consider using only one of those.  
  
## Proof of Concept  
Instances include:  
```
CDSTemplate.sol:287:    function totalLiquidity() public view returns (uint256 _balance) {
IndexTemplate.sol:491:    function leverage() public view returns (uint256 _rate) {
IndexTemplate.sol:504:    function totalLiquidity() public view returns (uint256 _balance) {
PoolTemplate.sol:628:        returns (uint256 premium)
PoolTemplate.sol:833:        returns (uint256 _balance)
PoolTemplate.sol:846:    function utilizationRate() public view override returns (uint256 _rate) {
PoolTemplate.sol:858:    function totalLiquidity() public view override returns (uint256 _balance) {
PoolTemplate.sol:866:    function originalLiquidity() public view returns (uint256 _balance) {
``` 
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Remove the unused named returns


