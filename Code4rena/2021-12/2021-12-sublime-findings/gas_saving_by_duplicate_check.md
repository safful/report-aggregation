## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas saving by duplicate check](https://github.com/code-423n4/2021-12-sublime-findings/issues/5) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept
In the contract `StrategyRegistry` the method `initialize` execute a require in order to check that the `_maxStrategies` is different than 0, this check will be done later inside the method `_updateMaxStrategies`, so it's duplicated and can be removed.

## Tools Used
Manual review

## Recommended Mitigation Steps
Remove the _maxStrategies checks inside the initialize method.

