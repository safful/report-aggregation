## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: Costly operations inside a loop (`IndexTemplate._adjustAlloc()`)](https://github.com/code-423n4/2022-01-insure-findings/issues/69) 

# Handle

Dravee


# Vulnerability details

## Impact
Repetitive and expensive SSTORE opcode operations inside loops

## Proof of Concept
```
	totalAllocatedCredit -= _available (contracts/IndexTemplate.sol#368)
	totalAllocatedCredit -= _decrease (contracts/IndexTemplate.sol#395)
	totalAllocatedCredit += _allocate (contracts/IndexTemplate.sol#401)
```

## Tools Used
Slither

## Recommended Mitigation Steps
Create a memory variable which will be used to compute a `_totalAllocatedCredit` that will get added to `totalAllocatedCredit` storage variable outside the loop.
As an idea, you could create 1 such `int` variable and use it's value after the for-loop, or you could create 2 uint variables where 1 would store the _totalDecrease and 1 would store the _totalAllocate, and respectively substract and add them.


