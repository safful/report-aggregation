## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [setupParticipant() function does not check for zero address](https://github.com/code-423n4/2021-11-malt-findings/issues/130) 

# Handle

jayjonah8


# Vulnerability details

## Impact
The setupParticipant() function in AuctionParticipant.sol does not have require statements to protect again contracts that do not yet exist.  It sets the addresses for " _impliedCollateralService", "_rewardToken", and "_auction" and can only be called once so its vital to have this guard in place.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AuctionParticipant.sol#L26

## Tools Used
Manual code review 

## Recommended Mitigation Steps
Add require checks for the addresses that are passed in the setupParticipant() function checking if they exist like: require("address" != address(0), "contract does not exist")

