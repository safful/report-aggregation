## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity
- resolved

# [Unused named return values are misleading and could lead to errors](https://github.com/code-423n4/2021-06-realitycards-findings/issues/96) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The code base uses a mix of named return values and explicit returns. In some places, the named return values are never assigned to and explicit returns are used instead. 

Impact: This makes code readability and auditability hard potentially leading to errors and missed vulnerabilities.

## Proof of Concept

Named return value shouldContinue is never assigned in _collectRentAction(): https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L856


Named return value didUpdateEverything is never assigned in _collectRent(): https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L1040

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove unassigned named return variables and be consistent in named vs explicit return usage.

