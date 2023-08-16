## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Functions can be external](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/262) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `withdrawAVAX()` function of LaunchEvent.sol and `initialize()` function of RocketJoeStaking.sol can be declared external for gas savings

## Proof of Concept

- [withdrawAVAX](https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L349)
- [initialize](https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeStaking.sol#L57)

## Recommended Mitigation Steps

Declare functions as external instead of public when possible

