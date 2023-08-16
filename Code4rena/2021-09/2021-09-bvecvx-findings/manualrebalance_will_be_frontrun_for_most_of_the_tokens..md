## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [ManualRebalance will be frontrun for most of the tokens.](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/5) 

# Handle

tensors


# Vulnerability details

## Impact
We have previously seen that the harvest function can be exploited for almost all the tokens at stake.
Since ManualRebalance calls harvest, it is also unsafe and funds swapped using it will likely be lost.

## Proof of Concept
https://github.com/code-423n4/2021-09-bvecvx/blob/1d64bd58c7a4224cc330cef283561e90ae6a3cf5/veCVX/contracts/veCVXStrategy.sol#L444-L453

## Recommended Mitigation Steps
Adding an amount out minimum here will work that should be passed on to the harvest method.

