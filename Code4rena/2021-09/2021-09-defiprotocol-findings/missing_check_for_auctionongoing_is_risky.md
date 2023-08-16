## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing check for auctionOngoing is risky](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/157) 

# Handle

0xRajeev


# Vulnerability details

## Impact

killAuction() is missing a require() to check that auctionOngoing == true before setting it to false. While currently, the caller publishNewIndex() in Basket has this condition checked, any other usages may accidentally call this when auction is not ongoing.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L43-L45

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L175-L187

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Add require(auctionOngoing == true)

