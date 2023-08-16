## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Open TODOs/questions](https://github.com/code-423n4/2021-10-mochi-findings/issues/139) 

# Handle

ye0lde


# Vulnerability details

## Impact
Open TODOs can point to programming or other errors that still need to be fixed.

## Proof of Concept

These are TODOs written as comments:
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/feePool/FeePoolV0.sol#L57
https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/vault/MochiVault.sol#L163

## Tools Used
VS Code

## Recommended Mitigation Steps
Resolve the TODOs/open questions.


