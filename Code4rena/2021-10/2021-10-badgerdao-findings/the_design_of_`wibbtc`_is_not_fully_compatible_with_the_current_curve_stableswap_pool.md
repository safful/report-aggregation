## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [The design of `wibBTC` is not fully compatible with the current Curve StableSwap pool](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/65) 

# Handle

WatchPug


# Vulnerability details

Per the documentation, `wibBTC` is designed for a Curve StableSwap pool. However, the design of `wibBTC` makes the balances change dynamically and automatically. This is unusual for an ERC20 token, and it's not fully compatible with the current Curve StableSwap pool.

Specifically, a Curve StableSwap pool will maintain the balances of its `coins` based on the amount of tokens added, removed, and exchanged each time. In another word, it can not adopt the dynamic changes of the balances that happened automatically.

The pool's actual dynamic balance of `wibBTC` will deviate from the recorded balance in the pool contract as the `pricePerShare` increases.

Furthermore, there is no such way in Curve StableSwap similar to the `sync()` function of UNI v2, which will force sync the stored `reserves` to match the balances.

### PoC

Given:

- The current `pricePerShare` is: `1`;
- The Curve pool is newly created with 0 liquidity;

1. Alice added `100 wibBTC` and `100 wBTC` to the Curve pool; Alice holds 100% of the pool;
2. After 1 month with no activity (no other users, no trading), and the `pricePerShare` of `ibBTC` increases to `1.2`;
3. Alice removes all the liquidity from the Curve pool.

While it's expected to receive `150 wibBTC` and `100 wBTC`, Alice actually can only receive `100 wibBTC` and `100 wBTC`.

### Recommendation

Consider creating a revised version of the Curve StableSwap contract that can handle dynamic balances properly.

