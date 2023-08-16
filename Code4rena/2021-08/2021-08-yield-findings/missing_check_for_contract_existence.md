## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Timelock

# [Missing check for contract existence](https://github.com/code-423n4/2021-08-yield-findings/issues/55) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Low-level call returns success even if the contract is non-existent. This requires a contract existence check before making the low-level call.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/TimeLock.sol#L93

See: “The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.” from https://docs.soliditylang.org/en/v0.8.7/control-structures.html#error-handling-assert-require-revert-and-exceptions

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check for target contract existence before call.

