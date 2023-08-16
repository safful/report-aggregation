## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Missing event for critical operation of new Liquidity locker creation in LiquidityLockerFactory.sol](https://github.com/code-423n4/2021-04-maple-findings/issues/31) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Lockers are critical contracts that hold custody of different Maple assets. While there are five lockers created initially (as described in https://github.com/maple-labs/maple-core/wiki/Lockers), such new lockers created by the factory should be logged as events for off-chain monitoring, similar to whats done in StakeLocker. However, such an event emission is missing here.

## Proof of Concept

https://github.com/maple-labs/maple-core/blob/355141befa89c7623150a83b7d56a5f5820819e9/contracts/LiquidityLockerFactory.sol#L19-L24

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Create and emit a suitable event to log locker creation.


