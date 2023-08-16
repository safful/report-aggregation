## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Inaccurate comment in `recoverTokens`](https://github.com/code-423n4/2021-11-streaming-findings/issues/213) 

# Handle

cmichel


# Vulnerability details

The `recoverTokens` function's comment states that the excess deposit tokens are `balance - depositTokenAmount`:

>     *      1. if its deposit token:
>     *          - DepositLock is fully done
>     *          - There are excess deposit tokens (balance - depositTokenAmount)

But it is `balance - (depositTokenAmount - redeemedDepositTokens)` where `(depositTokenAmount - redeemedDepositTokens)` is the outstanding redeemable amount.

## Impact
The code is correct.

## Recommended Mitigation Steps
Fix the comment.

