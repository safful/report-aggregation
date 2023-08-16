## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Changing function visibility from public to external/internal/private can save gas](https://github.com/code-423n4/2021-06-gro-findings/issues/37) 

# Handle

0xRajeev


# Vulnerability details

## Impact

For public functions, the input parameters are copied to memory automatically which costs gas. If a function is only called externally, making its visibility as external will save gas because external function’s parameters are not copied into memory and are instead read from calldata directly. If a function is called only from with that contract or derived contracts, making it internal/private can further reduce gas costs because the expensive calls are converted into cheaper jumps.


## Proof of Concept

The only callers of eoaOnly() are external contracts DepositHandler and WithdrawHandler.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L268
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/DepositHandler.sol#L112
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L211
The only caller of calcSystemTargetDelta() is Insurance.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Allocation.sol#L62-L63
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L213
The only caller of calcVaultTargetDelta() is Insurance.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Allocation.sol#L92-L93
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/insurance/Insurance.sol#L432
validGTokenDecrease() can be made private just like validGTokenIncrease because it is only called from within Controller.
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L448
https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L248


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change function visibility from public to external/private where possible.

