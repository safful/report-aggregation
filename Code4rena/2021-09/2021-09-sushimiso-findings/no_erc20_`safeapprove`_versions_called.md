## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [No ERC20 `safeApprove` versions called](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/99) 

# Handle

cmichel


# Vulnerability details

Some tokens don't correctly implement the EIP20 standard and their `approve` function returns `void` instead of a success boolean. Calling these functions with the correct EIP20 function signatures will always revert.

Calls to `.approve` with user-defined tokens are made in:
- `MISOLauncher.createLauncher`
- `MISOMarket.createMarket`

## Impact
Tokens that don't correctly implement the latest EIP20 spec, like USDT, will be unusable in the mentioned contracts as they revert the transaction because of the missing return value.

## Recommended Mitigation Steps
We recommend using OpenZeppelin’s `SafeERC20` versions with the `safeApprove` function that handle the return value check as well as non-standard-compliant tokens.

