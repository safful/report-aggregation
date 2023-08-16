## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [Burning does not update reserves](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/51) 

# Handle

cmichel


# Vulnerability details

The `ConcentratedLiquidityPool.burn` function sends out `amount0`/`amount1` tokens but only updates the reserves by decreasing it by the **fees of these amounts**.

```solidity
unchecked {
    // @audit decreases by fees only, not by amount0/amount1
    reserve0 -= uint128(amount0fees);
    reserve1 -= uint128(amount1fees);
}
```

This leads to the pool having wrong reserves after any `burn` action.
The pool's balance will be much lower than the reserve variables.

## Impact
As the pool's actual balance will be much lower than the reserve variables, `mint`ing and `swap`ing will not work correctly either.
This is because of the `amount0Actual + reserve0 <= _balance(token0)` check in `mint` using a much higher `reserve0` amount than the actual balance (already including the transferred assets from the user). An LP provider will have to make up for the missing reserve decrease from `burn` and pay more tokens.

The same holds true for `swap` which performs the same check in `_updateReserves`.

The pool essentially becomes unusable after a `burn` as LPs / traders need to pay more tokens.

## Recommended Mitigation Steps
The reserve should be decreased by what is transferred out. In `burn`'s case this is `amount0` / `amount1`.


