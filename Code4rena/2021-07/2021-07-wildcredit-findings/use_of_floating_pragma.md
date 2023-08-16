## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use of Floating Pragma](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/99) 

# Handle

JMukesh


# Vulnerability details

## Impact

https://swcregistry.io/docs/SWC-103

## Proof of Concept
Most of  listed files uses floating pragma, some are

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/LPTokenMaster.sol#L6

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/Controller.sol#L3

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/InterestRateModel.sol#L6

## Tools Used

manual review

## Recommended Mitigation Steps
use fixed solidity version

