## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Hardcoded constants are risky](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/174) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Hardcoded constants in code is risky for auditability/readability/maintainability. The Factory contract uses 2e17 as a threshold check for ownerSplit instead of using a contract constant as done in other places.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L56

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Create a contract constant and use that as done in other places.

