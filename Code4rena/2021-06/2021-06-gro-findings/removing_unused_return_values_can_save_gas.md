## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Removing unused return values can save gas](https://github.com/code-423n4/2021-06-gro-findings/issues/41) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The investDelta return variable from function getVaultDeltaForDeposit() is ignored at the only call-site in DepositHandler. Removing it can save some gas.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L144-L152

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L171-L175

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L193


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove unused return value or add logic to use it at caller.

