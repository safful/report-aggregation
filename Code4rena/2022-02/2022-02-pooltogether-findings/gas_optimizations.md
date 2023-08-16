## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-pooltogether-findings/issues/38) 

### Using `bool`s for storage incurs overhead
```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27
```solidity
mapping(address => mapping(address => bool)) internal representatives;         
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/TWABDelegator.sol#L159


### Functions not called by the contract itself should be `external` rather than `public`
```solidity
function initialize(uint96 _lockUntil) public {           
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L29


### Using `> 0` costs more gas than `!= 0` when used on uints in a `require()` statement
```solidity
require(_amount > 0, "TWABDelegator/amount-gt-zero");            
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/TWABDelegator.sol#L601


### `<array>.length` should not be looked up in every loop of a for-loop
Even memory arrays incur the overhead of bit tests and bit shifts to calculate the array length
```solidity
for (uint256 i = 0; i < _data.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L33

```solidity
for (uint256 i = 0; i < calls.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L42


### `++i`/`i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in for- and while-loops
```solidity
for (uint256 i = 0; i < _data.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L33

```solidity
for (uint256 i = 0; i < calls.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L42


### `++i` costs less gas than `++i`, especially when it's used in for-loops (`--i`/`i--` too)
```solidity
for (uint256 i = 0; i < _data.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L33

```solidity
for (uint256 i = 0; i < calls.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L42


### It costs more gas to initialize variables to zero than to let the default of zero be applied
```solidity
for (uint256 i = 0; i < _data.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/PermitAndMulticall.sol#L33

```solidity
for (uint256 i = 0; i < calls.length; i++) {      
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L42


### It costs less gas to inline the code of functions that are only called once
Doing so saves the cost of allocating the function selector, and the cost of the jump when the function is called.
```solidity
  function _requireDelegatorNotZeroAddress(address _delegator) internal pure {
    require(_delegator != address(0), "TWABDelegator/dlgtr-not-zero-adr");
  }
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/TWABDelegator.sol#L608-L610


### Using stack copies of variables when repeated access is required uses less gas
Even memory arrays incur the overhead of bit tests and bit shifts to calculate the array length. `calls.length` can be cached
```solidity
    bytes[] memory response = new bytes[](calls.length);
    for (uint256 i = 0; i < calls.length; i++) {
```
https://github.com/pooltogether/v4-twab-delegator/blob/2b6d42506187dd7096043e2dfec65fa06ab18577/contracts/Delegation.sol#L41-L42
