## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Oracles

# [Two functions with same code can be replaced by a single one](https://github.com/code-423n4/2021-08-yield-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

As noted in the code comment, peek and get functions are the same for this oracle. So we can change `peek` to public visibility and have `get` call `peek` instead of copying the same code here. Minor deployment cost savings but increase in readability/maintainability.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/oracles/composite/CompositeMultiOracle.sol#L91

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/oracles/composite/CompositeMultiOracle.sol#L74-L128

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Replace two functions having the same code with a single function.

