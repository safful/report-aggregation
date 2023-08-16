## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Oracles

# [CTokenMultiOracle.sol - Add natspec documentation](https://github.com/code-423n4/2021-08-yield-findings/issues/11) 

# Handle

PierrickGT


# Vulnerability details

# CTokenMultiOracle.sol - Add natspec documentation

## Impact
In `CTokenMultiOracle.sol`, internal functions, event, struct, public constant and mapping are not documented.

Functions parameters and return values should also be documented for external functions.

Natspec documentation should also be added to describe what this contract is all about.

## Recommended Mitigation Steps
Add natspec documentation to describe the contract:
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L11

Add natspec documentation for the following code:
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L14
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L16
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L18
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L24
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L73
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L91
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L109

Add natspec documentation for parameters and return value of these functions:
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L29
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L36
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L51
- https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L64


