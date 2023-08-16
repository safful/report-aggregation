## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Inaccurate Revert Message](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/63) 

# Handle

leastwood


# Vulnerability details

## Impact

The `_decreaseUserTwab()` function is used to decrease an account's TWAB balance when Ticket tokens are transferred between users or delegated to other users. If the amount to decrease exceeds the account's TWAB balance, the function will revert. However, this message does not fully reflect the function's behaviour.

## Proof of Concept

https://github.com/pooltogether/v4-core/blob/master/contracts/Ticket.sol#L364

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider updating the aforementioned revert message to correctly the function behaviour instead of a generic message.

