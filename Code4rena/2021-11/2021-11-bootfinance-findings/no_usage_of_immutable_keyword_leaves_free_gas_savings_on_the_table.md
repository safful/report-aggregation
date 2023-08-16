## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [No usage of immutable keyword leaves free gas savings on the table](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/1) 

# Handle

TomFrench


# Vulnerability details

## Impact

Increased gas costs + risk of accidental changes to values expected to be fixed.

## Proof of Concept

Several contracts contain variables which are set at deploy time and never changed again. For example see `PublicSale.sol`

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L70-L81

Since solidity 0.6.5, variables can be marked `immutable` which avoids the need for SLOADs when reading these variables - decreasing gas costs and protecting against accidentally modifying these variables.

https://blog.soliditylang.org/2020/05/13/immutable-keyword/#:~:text=With%20version%200.6.,time%20of%20a%20deployed%20contract.

## Tools Used

Manual inspection

## Recommended Mitigation Steps

Inspect all contracts for variables which are set once and then never modified, apply `immutable` keyword and adjust constructors to not read these values (instead use passed parameters)

