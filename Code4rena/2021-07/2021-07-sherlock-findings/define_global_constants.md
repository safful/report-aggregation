## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Define Global Constants](https://github.com/code-423n4/2021-07-sherlock-findings/issues/30) 

# Handle

hickuphh3


# Vulnerability details

### Impact

For better code readability, it would be good to specify the constants `uint16(-1)`, `uint32(-1)` and `10**18` in a separate contract to be imported in relevant contracts.

- `10e17` was used in payout() instead of the conventional `10**18` defined everywhere else
- The docs specified that the cooldown fee and sherX weight are scaled by `10**18`, but they are scaled by `uint32(-1)` and `uint16(-1)` respectively (interfaces natspec is correct).

### Recommended Mitigation Steps

Consider suggestive constants like `MAX_SHERX_WEIGHT` or `SHERX_DENOM`, `MAX_COOLDOWN_FEE` or `COOLDOWN_FEE_DENOM` and `PRECISION`.

