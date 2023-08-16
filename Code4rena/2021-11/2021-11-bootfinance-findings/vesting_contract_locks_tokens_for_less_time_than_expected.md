## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Vesting contract locks tokens for less time than expected](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/82) 

# Handle

TomFrench


# Vulnerability details

## Impact

Tokens are locked for 1 day less than specified in spec.

## Proof of Concept

The vesting period is calculated here in `unixYear`

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L30

This results in a lockup of 364 days rather than the expected 365.

## Recommended Mitigation Steps

Replace line with `uint256 constant private unixYear = 365 days;`


