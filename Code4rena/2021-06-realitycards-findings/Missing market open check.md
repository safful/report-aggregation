## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Missing market open check](https://github.com/code-423n4/2021-06-realitycards-findings/issues/86) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Missing _checkState(States.OPEN) on first line of rentAllCards() as specified on L617. These core market functions are supposed to operate only when market is open but the missing check allows control to proceed further in the control flow. In this case, the function proceeds to call newRental() which has a conditional check state == States.OPEN and silently returns success otherwise, without reverting.

Impact: rentAllCards does not fail if executed when market is closed or locked. newRental returns silently without failure when market is closed or locked.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L617

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L637-L658

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L672


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add a require() to check market open state in the beginning of all core market functions and revert with an informative error string otherwise.

