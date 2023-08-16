## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings of (100*loop-iteration-count) by caching _tokens.end() in _tokenTotalSupply()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/36) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The loop iteration in _tokenTotalSupply() ends when currentToken matches _tokens.end() where _tokens is a state variable.

Impact: Checking against the state variable for every iteration costs 100 gas per iteration. Even with only two controlled tokens (tickets & sponsorship), this costs 100 more than caching this in a local memory variable and using that within the while predicate.


## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L1059

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L177


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache _tokens.end() in a local memory variable before the loop and using that within the while predicate.

