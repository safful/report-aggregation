## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Internal functions can be private if the contract is not herited](https://github.com/code-423n4/2022-01-livepeer-findings/issues/62) 

# Handle

Dravee


# Vulnerability details

## Impact  
`private` functions are cheaper than `internal` functions.  
  
## Proof of Concept  
Several `internal` functions are in contracts that are never inherited.  
  
Their `internal` keywords are there:  
  
```
arbitrum-lpt-bridge\contracts\L1\gateway\L1LPTGateway.sol:170:        internal
arbitrum-lpt-bridge\contracts\L1\gateway\L1Migrator.sol:505:    ) internal view {
arbitrum-lpt-bridge\contracts\L1\gateway\L1Migrator.sol:518:        internal
arbitrum-lpt-bridge\contracts\L2\gateway\L2LPTGateway.sol:123:        internal
arbitrum-lpt-bridge\contracts\L2\gateway\L2Migrator.sol:307:    ) internal {
arbitrum-lpt-bridge\contracts\L2\pool\DelegatorPool.sol:95:    function transferBond(address _delegator, uint256 _stake) internal {
arbitrum-lpt-bridge\contracts\L2\pool\DelegatorPool.sol:106:    function pendingStake() internal view returns (uint256) {
arbitrum-lpt-bridge\contracts\L2\pool\DelegatorPool.sol:110:    function pendingFees() internal view returns (uint256) {
protocol\contracts\Manager.sol:48:    function _onlyController() internal view {
protocol\contracts\Manager.sol:52:    function _onlyControllerOwner() internal view {
protocol\contracts\Manager.sol:56:    function _whenSystemNotPaused() internal view {
protocol\contracts\Manager.sol:60:    function _whenSystemPaused() internal view {  
``` 
  
Therefore, their visibility should be reduced to `private`.  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Define these functions as `private`.


