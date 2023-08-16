## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- disagree with severity
- resolved

# [Redundant allowance and balance checks](https://github.com/code-423n4/2021-06-realitycards-findings/issues/93) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In Market sponsor() the call to treasury.checkSponsorship() checks allowance and balance  of user. This is redundant because the call to treasury.sponsor downstream checks allowance again and insufficient balance would cause any transfer to fail anyway.

Impact: Given the gas sensitivity of the code base, removing this redundant check could help conserve gas and prevent any DoS from breaking gas limits.

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L810

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L386-L396

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L474-L478


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove redundant checks.

