## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [TRANSMUTATION_PERIOD Issues](https://github.com/code-423n4/2021-11-yaxis-findings/issues/116) 

# Handle

ye0lde


# Vulnerability details

## Impact

Using existing local variables instead of reading state variables will save gas by converting SLOADs to MLOADs. 

## Proof of Concept

newTransmutationPeriod can be used here instead of TRANSMUTATION_PERIOD :
https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/Transmuter.sol#L194

TRANSMUTATION_PERIOD is named like a constant when it is actually an updatable state variable.

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

Use the local variable instead of the state variable.
Rename TRANSMUTATION_PERIOD appropriately.

