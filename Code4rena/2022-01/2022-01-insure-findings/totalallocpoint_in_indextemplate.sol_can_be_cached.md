## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [totalAllocPoint in IndexTemplate.sol can be cached](https://github.com/code-423n4/2022-01-insure-findings/issues/140) 

# Handle

p4st13r4


# Vulnerability details

## Impact

`totalAllocPoint` in `set()` function is read several times from storage. It can be assigned to a local variable so the function is less expensive overall

## Proof of Concept

[https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L612](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/IndexTemplate.sol#L612)

## Tools Used

Editor

## Recommended Mitigation Steps

Assign `totalAllocPoint` to `localTotalAllocPoint` (or `cachedTotalAllocPoint`)

