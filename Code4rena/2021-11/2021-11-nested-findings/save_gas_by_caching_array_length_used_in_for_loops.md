## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Save gas by caching array length used in for loops](https://github.com/code-423n4/2021-11-nested-findings/issues/7) 

# Handle

0x0x0x


# Vulnerability details

## Proof of Concept
Example:
```
for (uint i = 0; i < arr.length; i++) {
//Operations not effecting the length of the array.
}
```
Loading length of array costs gas. Therefore, the length should be cached, if the length of the array doesn't change inside the loop.
Recommended implementation:
```
uint length = arr.length;
for (uint i = 0; i < length; i++) {
//Operations not effecting the length of the array.
}
```
By doing so the length is only loaded once rather than loading it as many times as iterations (Therefore, less gas is spent).

## Occurences
```
contracts/FeeSplitter.sol:108:        for (uint256 i = 0; i < _accounts.length; i++) {
contracts/FeeSplitter.sol:125:        for (uint256 i = 0; i < _tokens.length; i++) {
contracts/FeeSplitter.sol:210:        for (uint256 i = 0; i < shareholders.length; i++) {
contracts/FeeSplitter.sol:227:        for (uint256 i = 0; i < shareholders.length; i++) {
contracts/MixinOperatorResolver.sol:32:        for (uint256 i = 0; i < requiredAddresses.length; i++) {
contracts/MixinOperatorResolver.sol:48:        for (uint256 i = 0; i < requiredAddresses.length; i++) {
contracts/NestedFactory.sol:203:        for (uint256 i = 0; i < tokens.length; i++) {
contracts/NestedFactory.sol:280:        for (uint256 i = 0; i < _orders.length; i++) {
contracts/NestedFactory.sol:316:        for (uint256 i = 0; i < _orders.length; i++) {
contracts/OperatorResolver.sol:33:        for (uint256 i = 0; i < names.length; i++) {
contracts/OperatorResolver.sol:45:        for (uint256 i = 0; i < names.length; i++) {
contracts/OperatorResolver.sol:56:        for (uint256 i = 0; i < destinations.length; i++) {
```

