## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant `return` for named returns](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/151) 

# Handle

WatchPug


# Vulnerability details

Redundant code increase contract size and gas usage at deployment.

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L227-L233

```solidity
function calculateAddQuoteTokenLiquidityQuantities(
    uint256 _quoteTokenQtyDesired,
    uint256 _quoteTokenQtyMin,
    uint256 _baseTokenReserveQty,
    uint256 _totalSupplyOfLiquidityTokens,
    InternalBalances storage _internalBalances
) public returns (uint256 quoteTokenQty, uint256 liquidityTokenQty) {
```

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L282-L282

```solidity
return (quoteTokenQty, liquidityTokenQty);
```

L282, `return (quoteTokenQty, liquidityTokenQty)` is redundant.


