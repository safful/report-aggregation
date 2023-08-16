## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Oracles

# [CompositeMultiOracle.sol - Add natspec documentation](https://github.com/code-423n4/2021-08-yield-findings/issues/15) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In `CompositeMultiOracle.sol`, internal functions, event, struct, public constant and mapping are not documented.

Functions parameters and return values should also be documented for external functions.

Natspec documentation should also be added to describe what this contract is all about.

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L79-L81

https://github.com/code-423n4/2021-08-yield/blob/1383f6a715657547603cddd0fed824cde631c7db/contracts/oracles/compound/CTokenMultiOracle.sol#L97-L99

## Recommended Mitigation Steps
Add natspec documentation to describe the contract and not just his title:
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L11-L12

Add natspec documentation for the following code:
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L15
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L17-L18
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L20
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L25-L26
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L110
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L120
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L130
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L140

Add natspec documentation for parameters and return value of these functions:
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L31
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L38
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L52
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L59
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L74
- https://github.com/code-423n4/2021-08-yield/blob/e4227756bd2b74d683525d61f0636bed64325955/contracts/oracles/composite/CompositeMultiOracle.sol#L94

