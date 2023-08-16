## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Redundant if statements in market deployment function](https://github.com/code-423n4/2022-01-insure-findings/issues/41) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact
gas costs

## Proof of Concept

Here if the lengths of these arrays are zero we'll fall straight through the for loops so there's no need for the if statements.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L175-L191

## Recommended Mitigation Steps

Remove if statements

