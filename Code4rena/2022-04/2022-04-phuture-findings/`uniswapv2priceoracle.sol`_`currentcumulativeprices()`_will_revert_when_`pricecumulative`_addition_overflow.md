## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`UniswapV2PriceOracle.sol` `currentCumulativePrices()` will revert when `priceCumulative` addition overflow](https://github.com/code-423n4/2022-04-phuture-findings/issues/62) 

# Lines of code

https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/UniswapV2PriceOracle.sol#L62


# Vulnerability details

https://github.com/code-423n4/2022-04-phuture/blob/594459d0865fb6603ba388b53f3f01648f5bb6fb/contracts/UniswapV2PriceOracle.sol#L62

```solidity
(uint price0Cumulative, uint price1Cumulative, uint32 blockTimestamp) = address(pair).currentCumulativePrices();
```

Because the Solidity version used by the current implementation of `UniswapV2OracleLibrary.sol` is `>=0.8.7`, and there are some breaking changes in Solidity v0.8.0:

> Arithmetic operations revert on underflow and overflow. 

Ref: https://docs.soliditylang.org/en/v0.8.13/080-breaking-changes.html#silent-changes-of-the-semantics

While in `UniswapV2OracleLibrary.sol`, subtraction overflow is desired at `blockTimestamp - blockTimestampLast` in `currentCumulativePrices()`:

https://github.com/Uniswap/v2-periphery/blob/master/contracts/libraries/UniswapV2OracleLibrary.sol#L25-L33

```solidity
if (blockTimestampLast != blockTimestamp) {
    // subtraction overflow is desired
    uint32 timeElapsed = blockTimestamp - blockTimestampLast;
    // addition overflow is desired
    // counterfactual
    price0Cumulative += uint(FixedPoint.fraction(reserve1, reserve0)._x) * timeElapsed;
    // counterfactual
    price1Cumulative += uint(FixedPoint.fraction(reserve0, reserve1)._x) * timeElapsed;
}
```

In another word, `Uniswap/v2-periphery/contracts/libraries/UniswapV2OracleLibrary` only works at solidity < `0.8.0`.

As a result, when `price0Cumulative` or `price1Cumulative` is big enough, `currentCumulativePrices` will revert due to overflow.

### Impact

Since the overflow is desired in the original version, and it's broken because of using Solidity version >0.8. The `UniswapV2PriceOracle` contract will break when the desired overflow happens, and further breaks other parts of the system that relies on `UniswapV2PriceOracle`.

### Recommendation

Note: this recommended fix requires a fork of the library contract provided by Uniswap.

Change to:

```solidity
if (blockTimestampLast != blockTimestamp) {
    unchecked {
        // subtraction overflow is desired
        uint32 timeElapsed = blockTimestamp - blockTimestampLast;
        // addition overflow is desired
        // counterfactual
        price0Cumulative += uint(FixedPoint.fraction(reserve1, reserve0)._x) * timeElapsed;
        // counterfactual
        price1Cumulative += uint(FixedPoint.fraction(reserve0, reserve1)._x) * timeElapsed;
    }
}
```

