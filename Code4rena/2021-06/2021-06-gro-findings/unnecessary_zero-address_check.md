## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary zero-address check](https://github.com/code-423n4/2021-06-gro-findings/issues/35) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Unnecessary zero-address check for account in addReferral() because it is always msg.sender (can never be 0) in the only call site from DepositHandler::depositGToken(). Removing this check can save a little gas in the critical deposit flow.


## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L202

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L115

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove unnecessary zero-address check.

