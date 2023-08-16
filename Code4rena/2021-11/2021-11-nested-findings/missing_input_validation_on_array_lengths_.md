## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing input validation on array lengths ](https://github.com/code-423n4/2021-11-nested-findings/issues/103) 

# Handle

ye0lde


# Vulnerability details

## Impact

The functions below fail to perform input validation on arrays to verify the lengths match. 
A mismatch could lead to an exception or undefined behavior.

## Proof of Concept

names, destinations
https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/OperatorResolver.sol#L27-L39

_inputTokenAmounts, orders
https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/NestedFactory.sol#L321-L337

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

Add input validation to check that the length of both arrays match.

