## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching repeatedly read state variables in local variables can save gas](https://github.com/code-423n4/2021-06-gro-findings/issues/31) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Post-Berlin, SLOADs on state variables accessed first-time in a transaction increased from 800 gas to 2100, which is a 2.5x increase. Successive SLOADs cost 100 gas. Memory stores/loads (MSTOREs/MLOADs) cost only 3 gas. Therefore, by caching repeatedly read state variables in local variables within a function, one can save >=100 gas.

## Proof of Concept

* Caching ctrl address in a local variable will save 300 gas because it is SLOADed 4 times now in this critical deposit flow.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L112
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L115
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L119
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L121

* Caching lg address state variable in a local variable outside the loop can save 1100 gas by avoiding 4 unnecessary SLOADs per loop iteration (4*3 = 12 but one SLOAD is hoisted out of the loop = 11 extra SLOADS at 100 gas = 1100 gas).
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L147-L151

* Caching buoy address state variable in the function beginning can save 100 gas from an extra SLOAD.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L174-L176

* Caching insurance address in function beginning can save 100 gas from an unnecessary SLOAD.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L193
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L198

* Caching lg address in function beginning can save 100 gas from an unnecessary SLOAD.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L197
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L199

* Hoisting buoy state variable out of the loop and caching it in a local variable will save 300 gas from 3 unnecessary SLOADs.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L181

* Caching buoy in a local variable will save 100 gas.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L212
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L219

* Caching ctrl in a local variable at the function beginning and using that in the rest of this function will save 4 unnecessary SLOADs i.e. 400 gas in this function.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L211
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L221
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L226
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L236
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L260
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L264

* Hoisting buoy out of the loop and caching in a local variable will save 3 unnecessary SLOADs and so 300 gas.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L329

* Caching lg in a local variable will save 100 gas.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L356
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L357

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache repeatedly read state variables (especially those within a loop) in local variables at an appropriate part of the function (preferably the beginning) and use them instead of state variables. Converting SLOADs to MLOADs reduces gas from 100 to 3.

