## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [NestedReserve: Redundant valid token address checks](https://github.com/code-423n4/2021-11-nested-findings/issues/123) 

# Handle

GreyArt


# Vulnerability details

## Impact

The `transferFromFactory()` function is missing the `valid(address(_token))` modifier that is present in the `transfer()` and `withdraw()` functions.

It is in our opinion that these sanity checks on the token address are redundant, because the transaction will revert anyway in the SafeERC20 library.

## Recommended Mitigation Steps

Either add in the modifier check for the `transferFromFactory()` function. Alternatively, remove them from all the functions as a gas optimization.

