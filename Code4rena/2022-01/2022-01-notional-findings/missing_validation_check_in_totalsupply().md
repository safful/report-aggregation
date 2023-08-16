## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing validation check in totalSupply()](https://github.com/code-423n4/2022-01-notional-findings/issues/170) 

# Handle

SolidityScan


# Vulnerability details

## Description
The value of `totalSupply()` at https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L260
does not check if the value of totalSupply is 0 or not and it is per 

## Impact
The return value for the function `getPoolTokenShare` can be invalid because if there's an error in the `totalSupply()` the code at Line 260 will evaluate to divide by zero creating inconsistencies in the function logic.


## Proof of Concept
1. Check the function at https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L257-L261
2. At line 260 we will notice that the value of totalSupply() is directly being used to perform division to the multiplication of `bptBalance * sNOTEAmount`


## Recommended Mitigation Steps
Add a check if the value of `totalSupply()` is zero or not or some other edge cases that can cause inconsistencies. 

