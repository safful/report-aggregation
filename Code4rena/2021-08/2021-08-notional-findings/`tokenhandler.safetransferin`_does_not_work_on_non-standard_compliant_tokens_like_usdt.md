## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`TokenHandler.safeTransferIn` does not work on non-standard compliant tokens like USDT](https://github.com/code-423n4/2021-08-notional-findings/issues/80) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `TokenHandler.safeTransferIn` function uses the standard `IERC20` function for the transfer call and proceeds with a `checkReturnCode` function to handle non-standard compliant tokens that don't return a return value.
However, this does not work as calling `token.transferFrom(account, amount)` already reverts if the token does not return a return value, as `token`'s `IERC20.transferFrom` is defined to always return a `boolean`.

## Impact
When using any non-standard compliant token like USDT, the function will revert.
Withdrawals for these tokens are broken, which is bad as `USDT` is a valid underlying for the `cUSDT` cToken.

## Recommended Mitigation Steps
We recommend using [OpenZeppelin’s `SafeERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.1/contracts/token/ERC20/utils/SafeERC20.sol#L74) versions with the `safeApprove` function that handles the return value check as well as non-standard-compliant tokens.

