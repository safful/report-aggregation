## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`ConstantProductPool` mint liquidity computation should include fees](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/95) 

# Handle

cmichel


# Vulnerability details

The `ConstantProductPool` computes optimal balanced LP using `_nonOptimalMintFee`, which performs something like a swap.
The returned swap fees should be included in the "`k`" computation as `_handleFees` uses the growth in `k` to estimate the swap fees.

```solidity
// should not reduce fee0 and fee1
uint256 computed = TridentMath.sqrt((balance0 - fee0) * (balance1 - fee1));
```

> ⚠️ The same issue occurs in the `HybridPool.mint`.

## Impact
The non-optimal mint swap fees are not taken into account.

## Recommended Mitigation Steps
Compute `sqrt(k)` on the `balance0 * balance1` without deducting the swap fees.


