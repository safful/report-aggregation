## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache storage read and call results in the stack can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/33) 

# Handle

WatchPug


# Vulnerability details

Caching the result of `_registry[product].length()` can save gas from unnecessary extra SLOAD, function call, and code execution, especially in for loops.

Instances include:

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/incentivizer/Incentivizer.sol#L144-L144

```solidity=144
for (uint256 i; i < _registry[product].length(); i++) {
```


https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/incentivizer/Incentivizer.sol#L174-L174

```solidity=174
for (uint256 i; i < _registry[product].length(); i++) {
```


https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/incentivizer/Incentivizer.sol#L190-L190

```solidity=190
for (uint256 i; i < _registry[product].length(); i++) {
```

