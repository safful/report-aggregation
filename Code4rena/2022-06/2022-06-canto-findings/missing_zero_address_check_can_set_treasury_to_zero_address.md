## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Missing zero address check can set treasury to zero address](https://github.com/code-423n4/2022-06-canto-findings/issues/121) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L15-L20


# Vulnerability details

## Impact
AccountantDelegate.initialize() is missing a zero address check for `treasury_` parameter, which could may allow treasury to be mistakenly set to 0 address.

## Proof of Concept
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L20

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a require() check for zero address for the treasury parameter before changing the treasury address in the initialize function.

