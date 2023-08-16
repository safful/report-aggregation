## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ATokenYieldSource

# [Gas optimization on `redeemToken` of `ATokenYieldSource`](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/123) 

# Handle

shw


# Vulnerability details

## Impact

At line 213 of `ATokenYieldSource`, `depositToken()` can be replaced by `_tokenAddress()` to save gas since the former is a public function, while the latter is an internal function.

## Proof of Concept

Referenced code:
[ATokenYieldSource.sol#L213](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/yield-source/ATokenYieldSource.sol#L213)

## Recommended Mitigation Steps

Change `depositToken()` to `_tokenAddress()`.

