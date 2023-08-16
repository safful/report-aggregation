## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Mint library uses wrong error code for max collateral check](https://github.com/code-423n4/2022-01-timeswap-findings/issues/56) 

# Handle

hyh


# Vulnerability details

## Impact

There can be issues with troubleshooting and system usage analytics.

## Proof of Concept

E512 error code is meant for 'Debt is greater than max Debt' situation:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/ErrorCodes.md#e512


In Mint library E512 is used for collateral max value check:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/Mint.sol#L482


Also, code E513, that is to be used for collateral check above, is also used for max asset increase check, which doesn’t seem to have an error code of its own:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/Mint.sol#L481

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/contracts/libraries/Mint.sol#L643


## Recommended Mitigation Steps

Change Mint line 482 error code to be 513:
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Convenience/ErrorCodes.md#e513

Possibly add an error code for max asset check for Mint lines 481 and 643.

