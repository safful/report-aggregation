## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [IndexPool initial LP supply computation is wrong](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/78) 

# Handle

cmichel


# Vulnerability details

The `IndexPool.constructor` function already mints `INIT_POOL_SUPPLY = 100 * 1e18 = 1e20` LP tokens to the zero address.

When trying to use the pool, someone has to provide the actual initial reserve tokens in `mint`.
On the first `mint`, the pool reserves are zero and the token amount required to mint is just this `ratio` itself: `uint120 amountIn = reserve != 0 ? uint120(_mul(ratio, reserve)) : ratio;`

Note that the `amountIn` is **independent of the token** which does not make much sense.
This implies that all tokens must be provided in equal "raw amounts", regardless of their decimals and value.

## POC
#### Issue 1
Imagine I want to create a DAI/WBTC pool.
If I want to initialize the pool with 100$ of DAI, `amountIn = ratio` needs to be `100*1e18=1e20` as DAI has 18 decimals.
However, I now also need to supply `1e20` of WBTC (which has 8 decimals) and I'd need to pay `1e20/1e8 * priceOfBTC`, over a quadrillion dollars to match it with the 100$ of DAI.

#### Issue 2
Even in a pool where all tokens have the same decimals and the same value, like `USDC <> USDT`, it leads to issues:
- Initial minter calls `mint` with `toMint = 1e20` which sets `ratio = 1e20 * 1e18 / 1e20 = 1e18` and thus `amountIn = 1e18` as well. The total supply increases to `2e20`.
- Second minter needs to pay **less** tokens to receive the same amount of `1e18` LP tokens as the first minter. This should never be the case. `toMint = 1e20` => `ratio = 1e20 * 1e18 / 2e20 = 0.5e18`. Then `amountIn = ratio * reserve / 1e18 = 0.5*reserve = 0.5e18`. They only pay half of what the first LP provider had to pay.

## Impact
It's unclear why it's assumed that the pool's tokens are all in equal value - this is _not_ a StableSwap-like pool.

Any pool that uses tokens that don't have the same value and share the same decimals cannot be used because initial liquidity cannot be provided in an economically justifiable way.

It also leads to issues where the second LP supplier has to pay **less tokens** to receive the exact same amount of LP tokens that the initial minter receives. They can steal from the initial LP provider by burning these tokens again.

## Recommended Mitigation Steps
Do not mint the initial token supply to the zero address in the constructor.

Do it like Uniswap/Balancer and let the first liquidity provider provide arbitrary token amounts, then mint the initial pool supply.
If `reserve == 0`, `amountIn` should just take the pool balances that were transferred to this account.

In case the initial mint to the zero address in the constructor was done to prevent the "Uniswap-attack" where the price of a single wei of LP token can be very high and price out LPs, send a small fraction of this initial LP supply (~1000) to the zero address **after** it was minted to the first supplier in `mint`.


