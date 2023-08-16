## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Abstract contracts should really be interfaces](https://github.com/code-423n4/2021-09-swivel-findings/issues/93) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Interfaces are not allowed to define any functions while abstract contracts can have a few defined functions (with at least one undefined function). Abstract contracts declared in the project should really be interfaces because they do not define any functions. 

Keeping them abstract is risky because they allow defining functions that may be mistakenly exposed in inherited contracts. Interfaces by design prevent this security risk.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Abstracts.sol#L5-L40

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Convert contracts that do not define any functions to interfaces.

