## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Unnecessary `unchecked` keyword is used in `FYToken`](https://github.com/code-423n4/2021-05-yield-findings/issues/67) 

# Handle

shw


# Vulnerability details

## Impact

At line 172 in the contract `FYToken`, the `unchecked` keyword is unnecessary since no arithmetic operation is involved.

## Proof of Concept

Referenced code:
[FYToken.sol#L172](https://github.com/code-423n4/2021-05-yield/blob/main/contracts/FYToken.sol#L172)

## Recommended Mitigation Steps

Consider removing the `unchecked` keyword.

