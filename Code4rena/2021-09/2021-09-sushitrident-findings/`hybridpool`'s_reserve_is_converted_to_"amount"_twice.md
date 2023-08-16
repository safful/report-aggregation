## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`HybridPool`'s reserve is converted to "amount" twice](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/101) 

# Handle

cmichel


# Vulnerability details

The `HybridPool`'s reserves are stored as Bento "amounts" (not Bento shares) in `_updateReserves` because `_balance()` converts the current share balance to amount balances.
However, when retrieving the `reserve0/1` storage fields in `_getReserves`, they are converted to amounts a second time.

## Impact
The `HybridPool` returns wrong reserves which affects all minting/burning and swap functions.
They all return wrong results making the pool eventually economically exploitable or leading to users receiving less tokens than they should.

## POC
Imagine the current Bento amount / share price being `1.5`.
The pool's Bento _share_ balance being `1000`.
`_updateReserves` will store a reserve of `1.5 * 1000 = 1500`.
When anyone trades using the `swap` function, `_getReserves()` is called and multiplies it by `1.5` again, leading to using a reserve of 2250 instead of 1500.
A higher reserve for the output token leads to receiving more tokens as the swap output.
Thus the pool lost tokens and the LPs suffer this loss.

## Recommended Mitigation Steps
Make sure that the reserves are in the correct amounts.


