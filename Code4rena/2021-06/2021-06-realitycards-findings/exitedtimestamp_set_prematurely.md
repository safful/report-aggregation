## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [exitedTimestamp set prematurely](https://github.com/code-423n4/2021-06-realitycards-findings/issues/91) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The exitedTimestamp flag is used to prevent front-running of user exiting and re-entering in the same block. The setting of this flag in exit() should really be inside the conditionals and triggered only if current owner or if bidExists. It currently assumes that either of the two will always be true which may not necessarily be the case.

Impact: A user accidentally exiting a card he doesn't own or have a bid for currently will be marked as exited and prevented from a newRental in the same block. User can prevent one's own newRental from succeeding, because it was accidentally triggered, by front-running it himself with an exit. There could be other more realistic scenarios.


## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L784-L804

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L56-L57

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L678-L681


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Set exitedTimestamp flag only when the conditionals are true within exit()

