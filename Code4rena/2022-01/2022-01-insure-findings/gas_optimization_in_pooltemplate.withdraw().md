## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in PoolTemplate.withdraw()](https://github.com/code-423n4/2022-01-insure-findings/issues/59) 

# Handle

tqts


# Vulnerability details

## Impact
None

## Proof of Concept
The `withdrawalReq[msg.sender].timestamp` and `parameters.getLockup(msg.sender)` values are used twice in the `require` statements, and both times summed. 

## Tools Used
Manual review

## Recommended Mitigation Steps
Cache the sum value in a new variable. I've sent a similar report for IndexTemplate.withdraw() with a similar issue.

