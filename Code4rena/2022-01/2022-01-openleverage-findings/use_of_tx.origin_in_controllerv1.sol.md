## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Use of tx.origin in ControllerV1.sol](https://github.com/code-423n4/2022-01-openleverage-findings/issues/60) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In ControllerV1.sol in the updatePriceAllowed() function tx.origin is used.  tx.origin is a global variable in Solidity which returns the address of the account that sent the transaction. Using the variable could make a contract vulnerable if an authorized account calls into a malicious contract.

## Proof of Concept
https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/ControllerV1.sol#L163

https://swcregistry.io/docs/SWC-115

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Its recommended to use msg.sender instead

