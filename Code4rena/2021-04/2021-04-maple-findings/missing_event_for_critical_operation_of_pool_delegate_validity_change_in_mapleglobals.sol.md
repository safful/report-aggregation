## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Missing event for critical operation of Pool Delegate validity change in MapleGlobals.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/39) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Pool delegates are trusted actors (see https://github.com/maple-labs/maple-core/wiki/Security#trust-assumptions) and so any change (additions/removals) in their validity should be recorded for off-chain monitoring. However, such an event emission is missing here.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/MapleGlobals.sol#L232-L239

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Create and emit a suitable event to log pool delegate validity change in setPoolDelegateAllowlist function.


