## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Misplaced zero-address check](https://github.com/code-423n4/2021-06-realitycards-findings/issues/82) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Misplaced zero-address check for nfthub on L595 in createMarket() because nfthub cannot be 0 at this point as nfthub.addMarket() on L570 would have already reverted if that were the case.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L570

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCFactory.sol#L595

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Move nfthub zero-address check to before the call to nfthub.addMarket().

