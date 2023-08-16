## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Bonding doesn't work with fee-on transfer tokens](https://github.com/code-423n4/2021-11-malt-findings/issues/251) 

# Handle

cmichel


# Vulnerability details

Certain ERC20 tokens make modifications to their ERC20's `transfer` or `balanceOf` functions.
One type of these tokens is deflationary tokens that charge a certain fee for every `transfer()` or `transferFrom()`.

## Impact
The `Bonding._bond()` function will revert in the `_balanceCheck` when transferring a fee-on-transfer token as it assumes the entire `amount` was received.

## Recommended Mitigation Steps
To support fee-on-transfer tokens, measure the asset change right before and after the asset-transferring calls and use the difference as the actual bonded amount.

