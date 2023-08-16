## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Inconsistent usage of exponentiation for constants](https://github.com/code-423n4/2021-06-gro-findings/issues/83) 

# Handle

GalloDaSballo


# Vulnerability details

## Impact
Detailed description of the impact of this finding.
See Constants.sol, where you use 10**DECIMALS: https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/common/Constants.sol#L7

VS

FixedContracts.sol, where you use 1E6: https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/common/FixedContracts.sol#L13

While both expressions result in the same values, I recommend picking one to avoid potential confusion




