## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Wrong docs on UsdOracle](https://github.com/code-423n4/2021-04-maple-findings/issues/84) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `UsdOracle.sol` contract states:

> UsdOracle is a constant price oracle feed that always returns 1 USD in USDC precision.

The USDC precision is 6, but the oracle returns a precision of 8, so the comment does not match the code.


## Impact

A wrong precision on the oracle contract could lead to inflated/deflated prices.

## Recommended Mitigation Steps

It seems that the current contract code assumes a precision of 8 instead of 6 and works correctly.
Clarify if the documentation is wrong or the code needs to be updated.
If further development is done and the comment is assumed to be correct, one might use 100 times the actual USDC token balance.


