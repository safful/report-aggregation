## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing events for critical privileged functions](https://github.com/code-423n4/2021-11-nested-findings/issues/42) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Functions that are only executable by privileged users (e.g. onlyOwner) and have an impact (e.g. financial, trust) on other users should emit events.

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L94
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L103
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L166

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L74
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L79
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L271

and possible other

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Emit events for privileged actions

