## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ConstantProductPool & HybridPool: Adding and removing unbalanced liquidity yields slightly more tokens than swap](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/34) 

# Handle

GreyArt


# Vulnerability details

### Impact

A mint fee is applied whenever unbalanced liquidity is added, because it is akin to swapping the excess token amount for the other token.

However, the current implementation distributes the minted fee to the minter as well (when he should be excluded). It therefore acts as a rebate of sorts.

As a result, it makes adding and removing liquidity as opposed to swapping directly (negligibly) more desirable. An example is given below using the Constant Product Pool to illustrate this point. The Hybrid pool exhibits similar behaviour.

### Proof of Concept

1. Initialize the pool with ETH-USDC sushi pool amounts. As of the time of writing, there is roughly 53586.556 ETH and 165143020.5295 USDC.
2. Mint unbalanced LP with 5 ETH (& 0 USDC). This gives the user `138573488720892 / 1e18` LP tokens.
3. Burn the minted LP tokens, giving the user 2.4963 ETH and 7692.40 USDC. This is therefore equivalent to swapping 5 - 2.4963 = 2.5037 ETH for 7692.4044 USDC.
4. If the user were to swap the 2.5037 ETH directly, he would receive 7692.369221 (0.03 USDC lesser).

### Recommended Mitigation Steps

The mint fee should be distributed to existing LPs first, by incrementing `_reserve0` and `_reserve1` with the fee amounts. The rest of the calculations follow after.

ConstantProductPool

```jsx
(uint256 fee0, uint256 fee1) = _nonOptimalMintFee(amount0, amount1, _reserve0, _reserve1);
// increment reserve amounts with fees
_reserve0 += uint112(fee0);
_reserve1 += uint112(fee1);
unchecked {
    _totalSupply += _mintFee(_reserve0, _reserve1, _totalSupply);
}
uint256 computed = TridentMath.sqrt(balance0 * balance1);
...
kLast = computed;
```

HybridPool

```jsx
(uint256 fee0, uint256 fee1) = _nonOptimalMintFee(amount0, amount1, _reserve0, _reserve1);
// increment reserve amounts with fees
_reserve0 += uint112(fee0);
_reserve1 += uint112(fee1);
uint256 newLiq = _computeLiquidity(balance0, balance1);
...
```

