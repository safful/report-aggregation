## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Duplicated code in `unbond` and `unbondAndBreak` functions](https://github.com/code-423n4/2021-11-malt-findings/issues/178) 

# Handle

nathaniel


# Vulnerability details

## Impact
A large portion of the `unbond` and `unbondAndBreak` code of Bonding.sol is the same, to reduce code bloat and gas when calling the contract 

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Bonding.sol#L97-L109

## Tools Used
Manual

## Recommended Mitigation Steps
I would suggest wrapping the duplicated code into an internal function called by `unbond` and `unbondAndBreak`.

