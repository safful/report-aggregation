## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Migration's `leave` function allows leaving a committed proposal](https://github.com/code-423n4/2022-07-fractional-findings/issues/379) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/main/src/modules/Migration.sol#L141


# Vulnerability details

The `leave` function allows to leave a proposal even if the proposal has been committed and failed.
This makes it a (probably unintended) duplicate functionality of `withdrawContributions`, which is the function that should be used to withdraw failed contributions.

## Impact
User assets might be lost:
When withdrawing assets from a failed migration, users should get back a different amount of assets, according to the buyout auction result. (I detailed this in another issue - "Migration::withdrawContribution falsely assumes that user should get exactly his original contribution back").
But when withdrawing assets from a proposal that has not been committed, users should get back their original amount of assets, as that has not changed.
Therefore, if `leave` does not check if the proposal has been committed, users could call `leave` instead of `withdrawContribution` and get back a different amounts of assets than they deserve, on the expense of other users.

## Proof of Concept
The `leave` function [does not check](https://github.com/code-423n4/2022-07-fractional/blob/main/src/modules/Migration.sol#L141) anywhere whether `proposal.isCommited == true`.
Therefore, if a user calls it after a proposal has been committed and failed,
it will continue to send him his original contribution back,
instead of sending him the adjusted amount that has been returned from Buyout.

## Recommended Mitigation Steps
Revert in `leave` if `proposal.isCommited == true`.
You might be also able to merge the functionality of `leave` and `withdrawContribution`, but that depends on how you will implement the fix for `withdrawContribution`.

