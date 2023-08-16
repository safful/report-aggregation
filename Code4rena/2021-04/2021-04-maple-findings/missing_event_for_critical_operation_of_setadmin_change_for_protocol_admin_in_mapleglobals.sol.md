## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Missing event for critical operation of setAdmin change for Protocol admin in MapleGlobals.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/38) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The protocol admin defined in MapleGlobals can pause/unpause all important functionalities of the protocol. This critical operation of admin status change in setAdmin function should be logged as an event for off-chain monitoring. However, such an event emission is missing here.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/MapleGlobals.sol#L148-L156

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Create and emit a suitable event to log admin status change in setAdmin function.


