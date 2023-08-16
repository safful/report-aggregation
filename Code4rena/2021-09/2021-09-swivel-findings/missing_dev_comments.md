## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing Dev Comments](https://github.com/code-423n4/2021-09-swivel-findings/issues/71) 

# Handle

leastwood


# Vulnerability details

## Impact

The `cTokenAddress()` getter function in `MarketPlace.sol` is missing relevant dev comments and appropriate matching syntax. `address a` does not correctly match the proper representation which is the underlying token, typically referenced as `address u`.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/v2/test/marketplace/MarketPlace.sol#L169-L171

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider adding relevant dev comments and updating `address a` -> `address u` to better reflect its meaning.

