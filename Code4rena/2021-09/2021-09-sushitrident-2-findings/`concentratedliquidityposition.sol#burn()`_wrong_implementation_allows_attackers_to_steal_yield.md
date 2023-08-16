## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`ConcentratedLiquidityPosition.sol#burn()` Wrong implementation allows attackers to steal yield](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/52) 

# Handle

WatchPug


# Vulnerability details

When a user calls `ConcentratedLiquidityPosition.sol#burn()` to burn their liquidity, it calls `ConcentratedLiquidityPool.sol#burn()` -> `_updatePosition()`:

https://github.com/sushiswap/trident/blob/c405f3402a1ed336244053f8186742d2da5975e9/contracts/pool/concentrated/ConcentratedLiquidityPool.sol#L525-L553

The `_updatePosition()` function will return `amount0fees` and `amount1fees` of the whole position with the `lower` and `upper` tick and send them to the `recipient` alongside the burned liquidity amounts.

## Proof of Concept

1. Alice minted $10000 worth of liquidity with `lower` and `upper` tick set to 99 and 199;
2. Alice accumulated $1000 worth of fee in token0 and token1;
3. The attacker can mint a small amount ($1 worth) of liquidity using the same `lower` and `upper` tick;
4. The attacker calls `ConcentratedLiquidityPosition.sol#burn()` to steal all the unclaimed yield with the ticks of (99, 199) include the $1000 worth of yield from Alice.

## Recommended Mitigation Steps

Consider making `ConcentratedLiquidityPosition.sol#burn()` always use `address(this)` as `recipient` in:

```solidity
position.pool.burn(abi.encode(position.lower, position.upper, amount, recipient, unwrapBento));
```

and transfer proper amounts to the user.

