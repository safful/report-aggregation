## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Removing unnecessary check can save gas in withdraw flow](https://github.com/code-423n4/2021-06-gro-findings/issues/38) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The minAmount <= amount check in _prepareForWithdrawalSingle() is an unnecessary check because the same check has already passed in both lg.withdrawSingleByLiquidity and lg.withdrawSingleByExchange. And there is no logic that changes the checked parameters between the earlier checks and this one.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/WithdrawHandler.sol#L361

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L224

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L268


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove unnecessary check.

