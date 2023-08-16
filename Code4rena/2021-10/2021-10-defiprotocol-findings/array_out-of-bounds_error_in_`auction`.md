## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Array out-of-bounds error in `Auction`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/31) 

# Handle

pants


# Vulnerability details

The function `Auction.withdrawBounty()` accept an argument called `bountyIds` and use it as indices to determine which elements in the `_bounties` array should be loaded and treated. However, this function don't check that the indices it receives as an argument actually fits the bounds of the `_bounties` array.

## Impact
If one of the indices exceed the array length, there will be a revert with no informative error message. The user wouldn't know what caused the revert.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Add an appropriate require statement to this function to validate that the given argument fits the `_bounties` array bounds.

