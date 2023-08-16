## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas saving in ShortLockupContract](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/133) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas saving.

## Proof of Concept
The variables `yetiToken` and `unlockTime` inside the `ShortLockupContract` contract are never modified, so it's better to use immutable to avoid storage access.

## Tools Used
Gas saving

## Recommended Mitigation Steps
Use immutable

