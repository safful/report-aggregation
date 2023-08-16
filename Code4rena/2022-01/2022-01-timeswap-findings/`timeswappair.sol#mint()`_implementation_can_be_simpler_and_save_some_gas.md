## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`TimeswapPair.sol#mint()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/154) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L157-L169

```solidity=157
if (pool.state.totalLiquidity == 0) {
    uint256 liquidityTotal = MintMath.getLiquidityTotal(xIncrease);
    liquidityOut = MintMath.getLiquidity(maturity, liquidityTotal, protocolFee);

    pool.state.totalLiquidity += liquidityTotal;
    pool.liquidities[factory.owner()] += liquidityTotal - liquidityOut;
} else {
    uint256 liquidityTotal = MintMath.getLiquidityTotal(pool.state, xIncrease, yIncrease, zIncrease);
    liquidityOut = MintMath.getLiquidity(maturity, liquidityTotal, protocolFee);

    pool.state.totalLiquidity += liquidityTotal;
    pool.liquidities[factory.owner()] += liquidityTotal - liquidityOut;
}
```

### Recommendation

Change to:

```solidity=157
uint256 liquidityTotal = pool.state.totalLiquidity == 0 ?
    MintMath.getLiquidityTotal(xIncrease) :
    MintMath.getLiquidityTotal(pool.state, xIncrease, yIncrease, zIncrease);
liquidityOut = MintMath.getLiquidity(maturity, liquidityTotal, protocolFee);

pool.state.totalLiquidity += liquidityTotal;
pool.liquidities[factory.owner()] += liquidityTotal - liquidityOut;
```

1. Avoiding code duplication;
2. Using the ternary operator to make the code shorter.

