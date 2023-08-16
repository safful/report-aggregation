## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`_handleUnderSpending` reverts if condition is false](https://github.com/code-423n4/2021-11-nested-findings/issues/183) 

# Handle

cmichel


# Vulnerability details

The `NestedFactory._handleUnderSpending` function implements a condition as `_amountToSpent - _amountSpent > 0` instead of `_amountToSpent > _amountSpent`.
The former reverts if `_amountSpent > _amountToSpent` while the latter doesn't.

It's unclear which behavior is preferred.

## Recommended Mitigation Steps
Think about if `_amountSpent > _amountToSpent` should revert or not. If not, the `if` condition can be rewritten as `_amountSpent > _amountToSpent` which would also save gas.

