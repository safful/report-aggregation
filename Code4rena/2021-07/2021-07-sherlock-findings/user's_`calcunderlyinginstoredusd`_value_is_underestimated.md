## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [User's `calcUnderlyingInStoredUSD` value is underestimated](https://github.com/code-423n4/2021-07-sherlock-findings/issues/144) 

# Handle

shw


# Vulnerability details

## Impact

The `calcUnderlyingInStoredUSD()` function of `SherX` should return `calcUnderlyingInStoredUSD(getSherXBalance())` instead of `calcUnderlyingInStoredUSD(sx20.balances[msg.sender])` since there could be SherX unallocated to the user at the time of the function call. A similar function, `calcUnderlying()`, calculates the user's underlying tokens based on the user's current balance plus the unallocated ones.

## Proof of Concept

Referenced code:
[SherX.sol#L141](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L141)

## Recommended Mitigation Steps

Change `sx20.balances[msg.sender]` to `getSherXBalance()` at line 141.

