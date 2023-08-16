## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Missing event for critical operation of setAdmin change in Pool.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/69) 

# Handle

0xRajeev


# Vulnerability details

## Impact

An admin of a Pool can call claim() and setLiquidytCap() along with the Pool Delegate. This critical operation of admin status change in setAdmin function should be logged as an event for off-chain monitoring. However, such an event emission is missing here.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/Pool.sol#L329-L337

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Create and emit a suitable event to log admin status change in setAdmin function.


