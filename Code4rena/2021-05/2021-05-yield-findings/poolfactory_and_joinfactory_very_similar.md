## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [PoolFactory and JoinFactory very similar](https://github.com/code-423n4/2021-05-yield-findings/issues/14) 

# Handle

gpersoon


# Vulnerability details

## Impact
PoolFactory and JoinFactory contain very similar but also relatively complicated code.
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/yieldspace/PoolFactory.sol
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/JoinFactory.sol

The risk is that future changes/improvements in one contract might not be updated in the other.

## Proof of Concept

## Tools Used
Editor

## Recommended Mitigation Steps
Consider refactoring the code where the core code is put in a library and reused from both of the contracts.


