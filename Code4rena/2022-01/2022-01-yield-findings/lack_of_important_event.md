## Tags

- bug
- 0 (Non-critical)
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Lack of important event](https://github.com/code-423n4/2022-01-yield-findings/issues/43) 

# Handle

0x1f8b


# Vulnerability details

## Impact
owner can change the source without any warning.

## Proof of Concept
The method `Cvx3CrvOracle.setSource` should emit an event in order to be able to detect this call by dapps.

## Tools Used
Manual review

## Recommended Mitigation Steps
Emit an event

