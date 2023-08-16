## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`_doSherX` does not return correct precision and it's confusing](https://github.com/code-423n4/2021-07-sherlock-findings/issues/108) 

# Handle

cmichel


# Vulnerability details

The `_doSherX` function does not return the correct precision of `sherUsd` and it is **not** the "Total amount of USD of the underlying tokens that are being transferred" that the documentation mentions.

```solidity
sherUsd = amounts[i].mul(sx.tokenUSD[tokens[i]]);
```

Instead, the amount is inflated by `1e18`, it should divide the amount by `1e18` to get a USD value with 18 decimal precision.

The severity is low as the calling site in `payout` makes up for it by dividing by `1e18` in the `deduction` computation.

We still recommend returning the correct amount in `_doSherX` already to match the documentation and avoid any future errors when using its unintuitive return value.

