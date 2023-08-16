## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Oracles

# [Redundant check ](https://github.com/code-423n4/2021-08-yield-findings/issues/47) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The require check for decimals_ <= 18 is unnecessary given its set to 18 right above unless this needs to be obtained differently as hinted by the comment.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/oracles/compound/CTokenMultiOracle.sol#L110-L111

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate check and remove.

