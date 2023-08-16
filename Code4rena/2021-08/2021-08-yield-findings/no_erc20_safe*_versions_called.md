## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- ERC20Rewards

# [No ERC20 safe* versions called](https://github.com/code-423n4/2021-08-yield-findings/issues/31) 

# Handle

cmichel


# Vulnerability details

The `claim` function performs an ERC20 transfer `rewardsToken.transfer(to, claiming);` but does not check the return value, nor does it work with all legacy tokens.

Some tokens (like USDT) don't correctly implement the EIP20 standard and their `transfer`/`transferFrom` function return `void` instead of a success boolean. Calling these functions with the correct EIP20 function signatures will always revert.

The `ERC20.transfer()` and `ERC20.transferFrom()` functions return a boolean value indicating success. This parameter needs to be checked for success.
Some tokens do **not** revert if the transfer failed but return `false` instead.


## Impact
Tokens that don't actually perform the transfer and return `false` are still counted as a correct transfer and tokens that don't correctly implement the latest EIP20 spec, like USDT, will be unusable in the protocol as they revert the transaction because of the missing return value.

## Recommended Mitigation Steps
We recommend using OpenZeppelin’s `SafeERC20` versions with the `safeTransfer` and `safeTransferFrom` functions that handle the return value check as well as non-standard-compliant tokens.


