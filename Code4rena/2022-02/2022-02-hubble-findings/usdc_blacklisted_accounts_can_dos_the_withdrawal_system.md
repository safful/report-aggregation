## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [USDC blacklisted accounts can DoS the withdrawal system](https://github.com/code-423n4/2022-02-hubble-findings/issues/76) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/VUSD.sol#L53-L67


# Vulnerability details

## Impact
DoS of USDC withdrawal system

## Proof of Concept
Currently, withdrawals are queued in an array and processed sequentially in a for loop.
However, a `safeTransfer()` to USDC blacklisted user will fail. It will also brick the withdrawal system because the blacklisted user is never cleared.

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/VUSD.sol#L53-L67

## Tools Used
Manual review

## Recommended Mitigation Steps
Possible solutions:
1st solution:
Implement 2-step withdrawals:
    - In a for loop, increase the user's amount that can be safely withdrawn.
    - A user himself withdraws his balance

2st solution:
Skip blacklisted users in a processWithdrawals loop

