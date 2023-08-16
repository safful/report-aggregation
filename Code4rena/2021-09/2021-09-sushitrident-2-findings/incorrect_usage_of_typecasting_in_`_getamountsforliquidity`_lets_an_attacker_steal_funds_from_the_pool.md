## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Incorrect usage of typecasting in `_getAmountsForLiquidity` lets an attacker steal funds from the pool](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/83) 

# Handle

broccoli


# Vulnerability details

## Impact

The `_getAmountsForLiquidity` function of `ConcentratedLiquidityPool` explicitly converts the result of `DyDxMath.getDy` and `DyDxMath.getDx` from type `uint256` to type `uint128`. The explicit casting without checking whether the integer exceeds the maximum number (i.e., `type(uint128).max`) could cause incorrect results being used. Specifically, an attacker could exploit this bug to mint a large amount of liquidity but only pay a little of `token0` or `token1` to the pool and effectively steal other's funds when burning his liquidity.

## Proof of Concept

1. Suppose that the current price is at the tick `500000`, an attacker calls the `mint` function with the following parameters:

```
mintParams.lower = 100000
mintParams.upper = 500000
mintParams.amount1Desired = (1 << 128) + 71914955423 # a carefully chosen number
mintParams.amount0Desired = 0
```

2. Since the current price is equal to the upper price, we have

```
_liquidity = mintParams.amount1Desired * (1 << 96) // (priceUpper - priceLower)
           = 4731732988155153573010127840
```

3. The amounts of `token0` and `token1` that the attacker has to pay is

```
amount0Actual = 0
amount1Actual = uint128(DyDxMath.getDy(_liquidity, priceLower, priceUpper, true))
              = uint128(_liquidity * (priceUpper - priceLower) // (1 << 96)) # round up
              = uint128(340282366920938463463374607456141861046)             # exceed the max
              = 24373649590                                                  # truncated
```

4. The attacker only pays `24373649590` of `token1` to get `4731732988155153573010127840` of the liquidity, which he could burn to get more `token1`. As a result, the attacker is stealing the funds from the pool and could potentially drain it.

Referenced code:
[ConcentratedLiquidityPool.sol#L480](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPool.sol#L480)
[concentratedPool/DyDxMath.sol#L15](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/libraries/concentratedPool/DyDxMath.sol#L15)
[concentratedPool/DyDxMath.sol#L30](https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/libraries/concentratedPool/DyDxMath.sol#L30)

## Recommended Mitigation Steps

Check whether the result of `DyDxMath.getDy` or `DyDxMath.getDx` exceeds `type(uint128).max` or not. If so, then revert the transaction. Or consider using the [`SafeCast` library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) from OpenZeppelin instead.

