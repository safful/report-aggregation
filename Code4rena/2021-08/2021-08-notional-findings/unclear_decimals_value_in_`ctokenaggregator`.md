## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unclear decimals value in `cTokenAggregator`](https://github.com/code-423n4/2021-08-notional-findings/issues/70) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `cTokenAggregator.decimals` value is set to `18` but `cTokens` only have `8` decimals. It's unclear what this `decimals` field refers to.

## Recommended Mitigation Steps
If it should refer to the `cToken` decimals, it's wrong and should be set to `8`.
This value is not used inside the contract but it's `public` and anyone can read it.



