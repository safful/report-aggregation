## Tags

- bug
- duplicate
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity
- Oracles

# [Uninitialized `updateTime` variables in `CompositeMultiOracle`](https://github.com/code-423n4/2021-08-yield-findings/issues/37) 

# Handle

shw


# Vulnerability details

## Impact

The `peek` and `get` functions of `CompositeMultiOracle` do not initialize the return variable `updateTime`, which is always 0 since the oldest timestamp is chosen and returned.

## Proof of Concept

Referenced code:
[CompositeMultiOracle.sol#L76](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/oracles/composite/CompositeMultiOracle.sol#L76)
[CompositeMultiOracle.sol#L96](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/oracles/composite/CompositeMultiOracle.sol#L96)

## Recommended Mitigation Steps

Handle the case when `updateTimeIn` is 0 in the private `_peek` and `_get` functions. If so, simply return `updateTimeOut`.

