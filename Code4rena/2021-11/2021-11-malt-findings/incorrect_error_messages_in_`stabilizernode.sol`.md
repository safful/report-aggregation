## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect error messages in `StabilizerNode.sol`](https://github.com/code-423n4/2021-11-malt-findings/issues/103) 

# Handle

pmerkleplant


# Vulnerability details

## Impact

The functions `setDefaultIncentive` and `setExpansionDamping` in `StabilizerNode.sol`
require their arguments to be non-zero, i.e. to be positive, as their argument
types are `uint`.

However, the error messages state that the arguments should not be non-negative.

See lines [406](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L406) and [417](https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L417).

## Recommended Mitigation Steps

Change the error messages to something like: "Must be above 0".

