## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack of `address(0)` check](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/26) 

# Handle

pmerkleplant


# Vulnerability details

The arguments of type `address` in the following functions miss a zero-check.

- `initialize()`
- `setPendingGovernance()`
- `setOracle()`

In the case of `setPendingGovernance()`, where a zero-address could be legitim, it
should be stated as such in the docs, or forbidden otherwise.

## Tools Used
slither

