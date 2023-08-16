## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unsecure Ownership Transfer](https://github.com/code-423n4/2021-12-perennial-findings/issues/12) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Lost the owner by human error.

## Proof of Concept
The modification process of an owner is a delicate process, since the governance of our contract and therefore of the project may be at risk, for this reason it is recommended to adjust the owner’s modification logic, to a logic that allows to verify that the new owner is in fact valid and does exist.

It's mandatory to create a logic of the owner’s modification where a new owner is proposed first, the owner accepts the proposal and, in this way, we make sure that there are no errors when writing the address of the new owner. 

Source reference:
- UOwnable.transferOwnership

## Tools Used
Manual review

## Recommended Mitigation Steps
Use an ACK method for approve the new owner.

