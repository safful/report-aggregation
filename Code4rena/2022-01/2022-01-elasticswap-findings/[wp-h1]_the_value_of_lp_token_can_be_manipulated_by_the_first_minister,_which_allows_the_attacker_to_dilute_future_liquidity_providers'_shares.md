## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [[WP-H1] The value of LP token can be manipulated by the first minister, which allows the attacker to dilute future liquidity providers' shares](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/145) 

# Handle

WatchPug


# Vulnerability details

For the first minter of an Exchange pool, the ratio of `X/Y` and the `totalSupply` of the LP token can be manipulated.

A sophisticated attacker can mint and burn all of the LP tokens but `1 Wei`, and then artificially create a situation of rebasing up by transferring baseToken to the pool contract. Then `addLiquidity()` in `singleAssetEntry` mode.

Due to the special design of `singleAssetEntry` mode, the value of LP token can be inflated very quickly.

As a result, `1 Wei` of LP token can be worthing a significate amount of baseToken and quoteToken.

Combine this with the precision loss when calculating the amount of LP tokens to be minted to the new liquidity provider, the attacker can turn the pool into a trap which will take a certain amount of cut for all future liquidity providers by minting fewer LP tokens to them.

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L493-L512

```solidity
} else {
    // this user will set the initial pricing curve
    require(
        _baseTokenQtyDesired > 0,
        "MathLib: INSUFFICIENT_BASE_QTY_DESIRED"
    );
    require(
        _quoteTokenQtyDesired > 0,
        "MathLib: INSUFFICIENT_QUOTE_QTY_DESIRED"
    );

    tokenQtys.baseTokenQty = _baseTokenQtyDesired;
    tokenQtys.quoteTokenQty = _quoteTokenQtyDesired;
    tokenQtys.liquidityTokenQty = sqrt(
        _baseTokenQtyDesired * _quoteTokenQtyDesired
    );

    _internalBalances.baseTokenReserveQty += tokenQtys.baseTokenQty;
    _internalBalances.quoteTokenReserveQty += tokenQtys.quoteTokenQty;
}
```

https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/libraries/MathLib.sol#L204-L212

```solidity
function calculateLiquidityTokenQtyForDoubleAssetEntry(
    uint256 _totalSupplyOfLiquidityTokens,
    uint256 _quoteTokenQty,
    uint256 _quoteTokenReserveBalance
) public pure returns (uint256 liquidityTokenQty) {
    liquidityTokenQty =
        (_quoteTokenQty * _totalSupplyOfLiquidityTokens) /
        _quoteTokenReserveBalance;
}
```

### PoC

Given:

- The `Pool` is newly created;
- The market price of `baseToken` in terms of `quoteToken` is `1`.

The attacker can do the following steps in one tx:

1. `addLiquidity()` with `2 Wei of baseToken` and `100e18 quoteToken`, received `14142135623` LP tokens;
2. `removeLiquidity()` with `14142135622` LP tokens, the Pool state becomes:
- totalSupply of LP tokens: 1 Wei
- baseTokenReserveQty: 1 Wei
- quoteTokenReserveQty: 7071067813 Wei
3. `baseToken.transfer()` 7071067812 Wei to the Pool contract;
4. `addLiquidity()` with no baseToken and `50e18 quoteToken`;
5. `swapBaseTokenForQuoteToken()` with `600000000000000 baseToken`, the Pool state becomes:
- totalSupply of LP tokens: 1 Wei
- quoteTokenReserveQty 591021750159032
- baseTokenReserveQty 600007071067801
6. `baseToken.transfer()` 999399992928932200 Wei to the Pool contract;
7. `addLiquidity()` with no baseToken and `1e18 quoteToken`, the Pool state becomes:
- totalSupply of LP tokens: 1 Wei
- quoteTokenReserveQty: 1000000000000000013
- quoteTokenReserveQty: 985024641638342212
- baseTokenDecay: 0

From now on, `addLiquidity()` with less than `1e18` of `baseToken` and `quoteToken` will receive `0` LP token due to precision loss.

The amounts can be manipulated to higher numbers and cause most future liquidity providers to receive fewer LP tokens than expected, and the attacker will be able to profit from it as the attacker will take a larger share of the pool than expected.

### Recommendation

Consider requiring a certain amount of minimal LP token amount (eg, 1e8) for the first minter and lock some of the first minter's LP tokens by minting ~1% of the initial amount to the factory address.

