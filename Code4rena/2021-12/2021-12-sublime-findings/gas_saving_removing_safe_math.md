## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas saving removing safe math](https://github.com/code-423n4/2021-12-sublime-findings/issues/6) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept
The method addStrategy inside StrategyRegistry do a require with safe math: `require(strategies.length.add(1) <= maxStrategies, "StrategyRegistry::addStrategy - Can't add more strategies");` is not possible to has a map that could lead in an integer overflow, so remove this `add` and use a regular +  will safe gas.

## Tools Used
Manual review

## Recommended Mitigation Steps
Remove safe math in this call

