## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Array out-of-bounds errors in `ETHPoolDelegator`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/38) 

# Handle

pants


# Vulnerability details

The functions `ETHPoolDelegator.balances()` and `ETHPoolDelegator.coins()` accept an argument called `i` and use it as an index to determine which element in the `_balances` / `_coins` array should be loaded and returned. However, these functions don't check that the index they receive as an argument actually fits the bounds of the array.

## Impact
If the index exceeds the array length, there will be a revert with no informative error message. The user wouldn't know what caused the revert.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Add an appropriate require statement to each of these functions to validate that the given argument fits the `_balances` / `_coins` array bounds.

