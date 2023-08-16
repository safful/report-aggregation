## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`CompoundToNotionalV2.notionalCallback` ERC20 return values not checked](https://github.com/code-423n4/2021-08-notional-findings/issues/68) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
Some tokens (like USDT) don't correctly implement the EIP20 standard and their `transfer`/`transferFrom` function return `void` instead of a success boolean. Calling these functions with the correct EIP20 function signatures will always revert.

See `CompoundToNotionalV2.notionalCallback`'s `IERC20(underlyingToken).transferFrom` call.

## Impact
Tokens that don't correctly implement the latest EIP20 spec, like USDT, will be unusable in the protocol as they revert the transaction because of the missing return value.
As there is a `cToken` with `USDT` as the underlying this issue directly applies to the protocol.

## Recommended Mitigation Steps
We recommend using OpenZeppelin’s `SafeERC20` versions with the `safeTransfer` and `safeTransferFrom` functions that handle the return value check as well as non-standard-compliant tokens.


