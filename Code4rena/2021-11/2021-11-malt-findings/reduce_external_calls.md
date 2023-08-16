## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Reduce external calls](https://github.com/code-423n4/2021-11-malt-findings/issues/208) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L167

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L219

Before call _distributeSupply function, it already get priceTarget,
But in _distributeSupply, it again call external call to get price target.
This will use higher gas.


## Tools Used
Manual

## Recommended Mitigation Steps
Send price target in _distributeSupply() function argument, and please review all duplicated external calls and optimize them.

