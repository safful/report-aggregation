## Tags

- bug
- duplicate
- 1 (Low Risk)
- sponsor confirmed
- Strategy

# [Missing emits for events](https://github.com/code-423n4/2021-08-yield-findings/issues/51) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Few events are missing emits which prevents the intended data from being observed easily by off-chain interfaces.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L48-L49

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add emits or remove event declarations.

