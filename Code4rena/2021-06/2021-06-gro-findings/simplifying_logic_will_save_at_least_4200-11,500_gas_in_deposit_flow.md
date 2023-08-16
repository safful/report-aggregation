## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Simplifying logic will save at least 4200-11,500 gas in deposit flow](https://github.com/code-423n4/2021-06-gro-findings/issues/32) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The feeToken logic is to account for tokens that may charge transfer fees and therefore require balance checks before/after transfers. For now, the only token that is programmed to potentially do so (in future, not currently) is USDT (neither DAI/USDC have this capability).

Impact: While this flexible future-proof logic is good design, this costs 3 SLOADs = 3*2100 = 6300 gas for reading the state variable feeToken 3 times (different index each time i.e. costs 2100, not 100) while the only token programmed for transfer fees is USDT (which has never charged fees). 2100 gas + two external token balance calls for USDT (2600*2 = 5200 gas + balance gas costs) >= total of 7300 gas for USDT and 4200 gas for other two tokens is perhaps expensive to support this future-proofing logic. However, from a security-perspective, it might be safer to leave this in here for USDT but remove checking for other two.

## Proof of Concept
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L145-L149

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L163-L167

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate removing it completely or hardcoding logic only for USDT index=2 to save gas.

