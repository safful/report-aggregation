## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Assignment Of Variable To Default (Identity.sol)](https://github.com/code-423n4/2021-10-ambire-findings/issues/17) 

# Handle

ye0lde


# Vulnerability details

## Impact

A variable is being assigned its default value which is unnecessary.
Removing the assignment will save gas when deploying.

## Proof of Concept

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/Identity.sol#L9

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Remove the assignment.

