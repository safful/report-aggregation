## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Initialize to default state is redundant](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/158) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/contracts/Exchange.sol#L27-L28

```solidity
MathLib.InternalBalances public internalBalances =
    MathLib.InternalBalances(0, 0, 0);
```

Initialize `internalBalances` to the default state is redundant.

Change to `MathLib.InternalBalances public internalBalances;` can make the code simpler and save some gas.

