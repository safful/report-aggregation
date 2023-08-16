## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H2] Transferring `quoteToken` to the exchange pool contract will cause future liquidity providers to lose funds](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/146) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, the amount of LP tokens to be minted when `addLiquidity()` is calculated based on the ratio between the amount of newly added `quoteToken` and the current wallet balance of `quoteToken` in the `Exchange` contract.

However, since anyone can transfer `quoteToken` to the contract, and make the balance of `quoteToken` to be larger than `_internalBalances.quoteTokenReserveQty`, existing liquidity providers can take advantage of this by donating `quoteToken` and make future liquidity providers receive fewer LP tokens than expected and lose funds.

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L578-L582

```solidity
liquidityTokenQty = calculateLiquidityTokenQtyForDoubleAssetEntry(
    _totalSupplyOfLiquidityTokens,
    quoteTokenQty,
    _quoteTokenReserveQty // IERC20(quoteToken).balanceOf(address(this))
);
```

### PoC

Given:

- The `Exchange` pool is new;

1. Alice `addLiquidity()` with `1e18 baseToken` and `1e18 quoteToken`, recived `1e18` LP token;
2. Alice transfer `99e18 quoteToken` to the `Exchange` pool contract;
3. Bob `addLiquidity()` with `1e18 baseToken` and `1e18 quoteToken`;
3. Bob `removeLiquidity()` with all the LP token in balance.

**Expected Results**: Bob recived `1e18 baseToken` and >= `1e18 quoteToken`.

**Actual Results**: Bob recived ~`0.02e18 baseToken` and ~`1e18 quoteToken`.

Alice can now `removeLiquidity()` and recive ~`1.98e18 baseToken` and ~`100e18 quoteToken`.

As a result, Bob suffers a fund loss of `0.98e18 baseToken`.

### Recommendation

Change to:

```solidity
liquidityTokenQty = calculateLiquidityTokenQtyForDoubleAssetEntry(
    _totalSupplyOfLiquidityTokens,
    quoteTokenQty,
    _internalBalances.quoteTokenReserveQty
);
```

