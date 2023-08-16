## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unsafe cast in ConcentratedLiquidityPool burn leads to attack](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/50) 

# Handle

cmichel


# Vulnerability details

The `ConcentratedLiquidityPool.burn` function performs an unsafe cast of a `uint128` type to a _signed_ integer.

```solidity
(uint256 amount0fees, uint256 amount1fees) = _updatePosition(msg.sender, lower, upper, -int128(amount));
```

Note that `amount` is chosen by the caller and when choosing `amount = 2**128 - 1`, this is interpreted as `0xFFFFFFFFF... = -1` as a signed integer. Thus `-(-1)=1` adds 1 liquidity unit to the position

This allows an attacker to not only mint LP tokens for free but as this is the `burn` function it also redeems token0/1 amounts according to the unmodified `uint128` `amount` which is an extremely large value.

## POC
I created this POC that implements a hardhat test and shows how to steal the pool tokens.

Choosing the correct `amount` of liquidity to burn and `lower, upper` ticks is not straight-forward because of two competing constraints:
1. the `-int128(amount)` must be less than `MAX_TICK_LIQUIDITY` (see `_updatePosition`). This drives the the `amount` up to its max value (as the max `uint128` value is -1 => -(-1)=1 is very low)
2. The redeemed `amount0, amount1` values must be less than the current pool balance as the transfers would otherwise fail. This drives the `amount` down. However, by choosing a smart `lower` and `upper` tick range we can redeem fewer tokens for the same liquidity.

This example shows how to steal 99% of the `token0` pool reserves:
https://gist.github.com/MrToph/1731dd6947073343cf6f942985d556a6

## Impact
An attacker can steal the pool tokens.

## Recommended Mitigation Steps
Even though Solidity 0.8.x is used, type casts do not throw an error.
A [`SafeCast` library](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) must be used everywhere a typecast is done.


