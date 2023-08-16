## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Short-circuiting in an if-statement](https://github.com/code-423n4/2022-01-insure-findings/issues/82) 

# Handle

Dravee


# Vulnerability details

## Impact
> The operators “||” and “&&” apply the common short-circuiting rules. This means that in the expression “f(x) || g(y)”, if “f(x)” evaluates to true, “g(y)” will not be evaluated even if it may have side-effects.

Source: https://docs.soliditylang.org/en/v0.5.4/types.html#booleans

## Proof of Concept
In `IndexTemplate.sol:withdrawable()`, there's an if-statement as such:
```
293:                         if (i == 0 || _availableRate < _lowestAvailableRate) {
```
Here, the condition `i == 0` is always evaluated and is always equal to `false` when `i > 0`, meaning here a total of `poolList.length - 1` evaluations are always evaluated to `false`.

It's best to reorder the conditions such as this condition doesn't get evaluated if `_availableRate < _lowestAvailableRate` is satisfied:
```
293:                         if (_availableRate < _lowestAvailableRate || i == 0 ) {
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Apply the refacto

