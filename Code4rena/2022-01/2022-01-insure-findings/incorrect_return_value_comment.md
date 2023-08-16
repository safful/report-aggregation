## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Incorrect return value comment](https://github.com/code-423n4/2022-01-insure-findings/issues/157) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The comment for the return value of the `getCDS()` function in Registry.sol is incorrectly copied from elsewhere, possibly the `confirmExistence()` function. The return value is an address, not a boolean. This is considered low risk based on C4's [risk ratings](https://docs.code4rena.com/roles/wardens/judging-criteria#estimating-risk-tl-dr).

## Proof of Concept

The problematic comment is from the `getCDS()` function [here](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Registry.sol#L99). It is an incorrect duplicate of the comment for the `confirmExistence()` function [found here](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Registry.sol#L113).

## Recommended Mitigation Steps

Replace the comment with something like `@return CDS contract address`, which is used to describe this value in the `setCDS()` function.

