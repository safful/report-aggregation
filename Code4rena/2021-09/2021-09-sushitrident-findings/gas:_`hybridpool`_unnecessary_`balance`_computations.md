## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: `HybridPool` unnecessary `balance` computations](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/102) 

# Handle

cmichel


# Vulnerability details

The `HybridPool.burn` function subtracts some computation from `balance0`/`balance1`, but the result is never used.

```solidity
balance0 -= _toShare(token0, amount0);
balance1 -= _toShare(token1, amount1);
```

## Recommended Mitigation Steps
Unless it is used as an underflow check, the computation should be removed the result is not used.


