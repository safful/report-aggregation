## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [OperatorHelpers.sol: function decodeDataAndRequire state mutability can be restricted to pure](https://github.com/code-423n4/2021-11-nested-findings/issues/48) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
OperatorHelpers.sol: function decodeDataAndRequire state mutability can be restricted to pure
We don't read any storage variables, only use the arguments therefore, it can be restricted to pure.

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/libraries/OperatorHelpers.sol#L45

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
set function state mutability to pure

