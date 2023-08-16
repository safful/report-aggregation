## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing consistent zero address checks](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/263) 

# Handle

sirhashalot


# Vulnerability details

## Impact

Some functions in RocketJoeFactory.sol have zero checks for setting specific state variables, but there zero address checks are not always applied. Setting some of these state variables to the zero address, whether intentional or not, can break the protocol functionality. Adding these checks consistently would prevent this scenario.

## Proof of Concept

The [constructor in RocketJoeFactory.sol](https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L53-L61) performs zero address checks before setting the router, factory, penaltyCollector, and rJoe state variables.  Later in the same contract, the functions `setRJoe()`, `setPenaltyCollector()`, `setRouter()`, and `setFactory()` [omit the same zero address checks](https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L159-L188) that were applied earlier. Since the issues that can be caused by setting these state variables to the zero address exist whether setting the value in the constructor or in the setter function, these checks should be applied consistently.

## Recommended Mitigation Steps

Add zero address checks in the setter functions for these state variables just like is done in the constructor. If it is determined that a zero check for any of these state variables is not needed, then the zero check can be removed from the RocketJoeFactory.sol constructor for consistency.

