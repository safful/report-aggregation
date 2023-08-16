## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`CompoundToNotionalV2.enableToken` ERC20 missing return value check](https://github.com/code-423n4/2021-08-notional-findings/issues/67) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `enableToken` function performs an `ERC20.approve()` call but does not check the `success` return value.
Some tokens do **not** revert if the approval failed but return `false` instead.

## Impact
Tokens that don't actually perform the approve and return `false` are still counted as a correct approve.

## Recommended Mitigation Steps
We recommend using [OpenZeppelin’s `SafeERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.1/contracts/token/ERC20/utils/SafeERC20.sol#L74) versions with the `safeApprove` function that handles the return value check as well as non-standard-compliant tokens.

