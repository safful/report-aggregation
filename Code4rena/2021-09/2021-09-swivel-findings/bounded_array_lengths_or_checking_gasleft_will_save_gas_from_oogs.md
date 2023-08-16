## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Bounded array lengths or checking gasleft will save gas from OOGs](https://github.com/code-423n4/2021-09-swivel-findings/issues/116) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Swivel initiate() and exit() functions accept unbounded arrays from users which may lead to OOG exceptions with insufficient gas sent in transaction.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L55-L77

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L209-L234

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Bounding array lengths or checking gasleft are a good idea to reduce risk of OOG and save user’s gas.

