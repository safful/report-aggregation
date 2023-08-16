## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Uncontrolled call to controller, which can be the zero address](https://github.com/code-423n4/2022-01-insure-findings/issues/94) 

# Handle

camden


# Vulnerability details

## Impact
The `utilize()` function can be called while the controller is the zero address. This will fail. A comment in the constructor says that the controller shouldn't be the zero address.

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L350

## Recommended Mitigation Steps
`utilize` should have a check to see if the controller is not the zero address (like `_unutilize`) and give an appropriate error message.

