## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Different coding style for same pattern: x += y and sometimes x = x + y](https://github.com/code-423n4/2021-11-nested-findings/issues/45) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact

The pattern of adding/subtracting a variable to/from another value is sometimes written as x += y; and sometimes as x = x + y; (x -= y; and sometimes x = x - y;)
The shorter version x += y;/x -= y; increases readability.

## Proof of Concept

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L241

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L256

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L229

and possible others

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
use the x += y; / x -= y; pattern 

