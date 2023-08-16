## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Require statement can be moved to start of function](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/19) 

# Handle

bw


# Vulnerability details

## Impact

A [require](https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Factory.sol#L74) statement in `Factory.sol` could be performed prior to an expensive cross contract call, reducing the amount of gas wasted if the validation fails.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Factory.sol#L74

## Tools Used

N/A

## Recommended Mitigation Steps

Move the require statement before `basketImpl.validateWeights(tokens, weights);`

