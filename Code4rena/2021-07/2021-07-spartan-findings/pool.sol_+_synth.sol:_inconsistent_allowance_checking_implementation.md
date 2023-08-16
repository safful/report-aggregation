## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Pool.sol + Synth.sol: Inconsistent Allowance Checking Implementation](https://github.com/code-423n4/2021-07-spartan-findings/issues/55) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The contract performs allowance checks for transfers in 2 ways:

1. Check allowance is greater than requested amount, revert otherwise. Then do allowance decrement. (Eg. in `transferFrom`)
2. Directly do the allowance decrement, will revert for underflow since sol 0.8.3 is used. (Eg. in `burnFrom`

It is best to stick to 1 method for consistency. For gas optimizations, the 2nd method is better, but the first provides more meaningful revert messages to aid debugging.

### Recommended Mitigation Steps

Commit to either method, not both.

