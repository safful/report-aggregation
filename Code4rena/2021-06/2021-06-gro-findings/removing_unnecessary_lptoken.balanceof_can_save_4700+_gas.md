## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Removing unnecessary lpToken.balanceOf can save 4700+ gas](https://github.com/code-423n4/2021-06-gro-findings/issues/43) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In LifeGuard3Pool (LG) deposit(), lp token balance is determined for the crv3pool.add_liquidity() call. Given that LG does not hold any lp tokens between txs, there is no need to determine and subtract lp token balance before and after the curve add liquidity call. Removing the call on L204 will save at least 2600+2100=4700 gas from the external call.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L204-L206

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove the call on L204 and just get the balance on L206 without any subtraction.

