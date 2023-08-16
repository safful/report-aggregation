## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unsafe call to `.decimals`](https://github.com/code-423n4/2021-05-yield-findings/issues/32) 

# Handle

cmichel


# Vulnerability details

The `FYToken.constructor` performs an external call to `IERC20Metadata(address(IJoin(join_).asset())).decimals()`.
This function was optional in the initial ERC-20 and might fail for old tokens that therefore did not implement it.

## Impact
FyTokens cannot be created for tokens that implemented the old initial ERC20 without the `decimals` function.

## Recommended Mitigation Steps
Consider using the helper function in the utils to retrieve it `SafeERC20Namer.tokenDecimals`, the same way the `Pool.constructor` works.

