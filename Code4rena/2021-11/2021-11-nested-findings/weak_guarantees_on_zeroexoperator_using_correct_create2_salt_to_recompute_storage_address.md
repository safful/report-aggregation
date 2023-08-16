## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Weak guarantees on ZeroExOperator using correct create2 salt to recompute storage address](https://github.com/code-423n4/2021-11-nested-findings/issues/4) 

# Handle

TomFrench


# Vulnerability details

## Impact

Potential for a broken deploy of operators which use a storage contract (not in the case of `ZeroExOperator` however)

## Proof of Concept

`ZeroExOperator` uses a create2 salt of `bytes32("nested.zeroex.operator")` to deploy its storage contract and this salt must be used to recompute this address in future.

It's then important to enforce that both steps use the same salt, however this is not strictly enforced. Currently a change to one must be manually updated in the other, if this was not done then calculation of the storage address would be incorrect.

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/operators/ZeroEx/ZeroExOperator.sol#L15

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/operators/ZeroEx/ZeroExOperator.sol#L60

This is not an issue in the current case but this is a potential footgun for future operators which use storage.

## Recommended Mitigation Steps

Place `bytes32("nested.zeroex.operator")` into a constant variable and use this variable instead.

