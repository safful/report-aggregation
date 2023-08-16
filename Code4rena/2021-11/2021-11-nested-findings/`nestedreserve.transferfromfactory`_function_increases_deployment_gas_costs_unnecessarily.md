## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`NestedReserve.transferFromFactory` function increases deployment gas costs unnecessarily](https://github.com/code-423n4/2021-11-nested-findings/issues/27) 

# Handle

TomFrench


# Vulnerability details

## Impact

`NestedReserve.transferFromFactory` is unused and so increases deployment costs for no gain

## Proof of Concept

`NestedReserve` has a `transferFromFactory` which can be seen not to be used in the codebase (and in the case the `NestedFactory` needs to send tokens to the reserve it can do so directly.)

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedReserve.sol#L55-L60

## Recommended Mitigation Steps

Remove this function.

