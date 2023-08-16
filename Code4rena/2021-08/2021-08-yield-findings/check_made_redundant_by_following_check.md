## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- EmergencyBrake
- Timelock

# [Check made redundant by following check](https://github.com/code-423n4/2021-08-yield-findings/issues/48) 

# Handle

0xRajeev


# Vulnerability details

## Impact
The check for array lengths is unnecessary in two places where the following check on txHash will anyway fail if the lengths don’t match with what was hashed earlier during schedule. Removing the length check can save a little gas.

Such a require() in cancel() can be removed because if there is a mismatch, the entry lookup in transactions[] will fail anyway and also, this will not be sceduled/executed.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/TimeLock.sol#L70-L72

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/TimeLock.sol#L81-L85


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate and remove these checks.

