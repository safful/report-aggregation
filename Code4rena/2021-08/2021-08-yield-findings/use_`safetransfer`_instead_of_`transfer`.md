## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed
- ERC20Rewards

# [Use `safeTransfer` instead of `transfer`](https://github.com/code-423n4/2021-08-yield-findings/issues/36) 

# Handle

shw


# Vulnerability details

## Impact

Tokens not compliant with the ERC20 specification could return `false` from the `transfer` function call to indicate the transfer fails, while the calling contract would not notice the failure if the return value is not checked. Checking the return value is a requirement, as written in the [EIP-20](https://eips.ethereum.org/EIPS/eip-20) specification:

> Callers MUST handle `false` from `returns (bool success)`. Callers MUST NOT assume that `false` is never returned!

## Proof of Concept

Referenced code:
[ERC20Rewards.sol#L175](https://github.com/code-423n4/2021-08-yield/blob/main/contracts/utils/token/ERC20Rewards.sol#L175)

## Recommended Mitigation Steps

Use the `SafeERC20` library [implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) from OpenZeppelin and call `safeTransfer` or `safeTransferFrom` when transferring ERC20 tokens.

