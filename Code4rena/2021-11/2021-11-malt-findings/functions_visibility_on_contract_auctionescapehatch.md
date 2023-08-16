## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [functions visibility on contract AuctionEscapeHatch](https://github.com/code-423n4/2021-11-malt-findings/issues/97) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability description
On contract **AuctionEscapeHatch** there is a function labeled as Public, However this function is not used within the contracts of **Malt** protocol.

## Impact
Improve coding style quality for developers and audit.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L94

## Tools Used
manual code review.

## Recommended Mitigation Steps
Evaluate functions labeled as public and set to external if needed just like the rest of functions inside this contract.

