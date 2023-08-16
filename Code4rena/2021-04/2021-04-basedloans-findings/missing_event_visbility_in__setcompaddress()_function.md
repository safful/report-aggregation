## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing event visbility in _setCompAddress() function](https://github.com/code-423n4/2021-04-basedloans-findings/issues/13) 

# Handle

toastedsteaksandwich


# Vulnerability details

## Impact
The _setCompAddress() function in the Comptroller contract does not emit an event when changing the comp address. While this does not impose any security risk, it does hinder a users ability to view any changes made to the comp address through the contract's lifetime. 

## Affected line
https://github.com/code-423n4/2021-04-basedloans/blob/main/code/contracts/Comptroller.sol#L1354

## Recommended Mitigation Steps
It is recommended to emit an event indicating the old comp address, and the new comp address to be used when calling the _setCompAddress() function. An example of such an event is `event NewCompAddress(address oldCompAddress, address newCompAddress)`.

