## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Lack of zero address validation](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/108) 

# Handle

JMukesh


# Vulnerability details

## Impact
Due to lack of zero address validation funds can be lost in following case

ex - No checking of address(0) in constructor
       No checking of address(0) while using low-level call to transfer eth

## Proof of Concept

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/TransferHelper.sol#L25

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/Controller.sol#L49

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/RewardDistribution.sol#L57

## Tools Used
manual review

## Recommended Mitigation Steps

add zero address validation

