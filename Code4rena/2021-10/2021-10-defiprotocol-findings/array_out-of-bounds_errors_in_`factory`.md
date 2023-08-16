## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Array out-of-bounds errors in `Factory`](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/30) 

# Handle

pants


# Vulnerability details

The functions `Factory.proposal()`, `Factory.getProposalWeights()` and `Factory.createBasket()` accept an argument called `proposalId`, `id` or `idNumber`, respectively, and use it as an index to determine which element in the `_proposals` array should be loaded and treated. However, these functions don't check that the index they receive as an argument actually fits the bounds of the `_proposals` array.

## Impact
If the index exceed the array length, there will be a revert with no informative error message. The user wouldn't know what caused the revert.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Add an appropriate require statement to each of these functions to validate that the given argument fits the `_proposals` array bounds.

