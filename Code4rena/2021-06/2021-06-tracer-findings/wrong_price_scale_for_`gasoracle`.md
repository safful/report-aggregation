## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [Wrong price scale for `GasOracle`](https://github.com/code-423n4/2021-06-tracer-findings/issues/93) 

# Handle

cmichel


# Vulnerability details

The `GasOracle` uses two chainlink oracles (GAS in ETH with some decimals, USD per ETH with some decimals) and multiplies their raw return values to get the gas price in USD.

However, the scaling depends on the underlying decimals of the two oracles and could be anything.
But the code assumes it's in 18 decimals.

> "Returned value is USD/Gas * 10^18 for compatibility with rest of calculations"

There is a `toWad` function that seems to involve scaling but it is never used.

## Impact**
If the scale is wrong, the gas price can be heavily inflated or under-reported. 

## Recommended Mitigation Steps
Check `chainlink.decimals()` to know the decimals of the oracle answers and scale the answers to 18 decimals such that no matter the decimals of the underlying oracles, the `latestAnswer` function always returns the answer in 18 decimals.

