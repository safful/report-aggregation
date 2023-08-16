## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Store hash of `type(ZeroExStorage).creationCode` rather than recalculating it on each call](https://github.com/code-423n4/2021-11-nested-findings/issues/35) 

# Handle

TomFrench


# Vulnerability details

## Impact

Deployment + runtime gas cost increase

## Proof of Concept

On each time we calculate the address of `ZeroExStorage` we hash the entirety of the creation code for `ZeroExStorage`. This means that not only do we have to perform a large hash operation over the entire creation bytecode of this contract, we need to store all of this bytecode in the `ZeroExOperator`'s deployed bytecode.

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/operators/ZeroEx/ZeroExOperator.sol#L61

This hash could be calculated once at deployment and then have this used cheaply each time, reducing both deployment and runtime costs.

## Recommended Mitigation Steps

Store `keccak256(type(ZeroExStorage).creationCode)` in an `immutable` (not `constant` as this still results in hashing being applied each time) variable.

