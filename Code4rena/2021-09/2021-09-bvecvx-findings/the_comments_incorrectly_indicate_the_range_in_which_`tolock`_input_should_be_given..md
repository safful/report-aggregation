## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [The comments incorrectly indicate the range in which `toLock` input should be given.](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/15) 

# Handle

tabish


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

The input `toLock` in the `manualRebalance` function should in terms of BPS else `toLock` should be changed accordingly in the function. The comments incorrectly indicate the range in which the input `toLock` should be given. https://github.com/code-423n4/2021-09-bvecvx/blob/1d64bd58c7a4224cc330cef283561e90ae6a3cf5/veCVX/contracts/veCVXStrategy.sol#L443

## Recommended Mitigation Steps
In the comments `toLock` should be = 10_000 as we are comparing with `MAX_BPS` https://github.com/code-423n4/2021-09-bvecvx/blob/1d64bd58c7a4224cc330cef283561e90ae6a3cf5/veCVX/contracts/veCVXStrategy.sol#L446

