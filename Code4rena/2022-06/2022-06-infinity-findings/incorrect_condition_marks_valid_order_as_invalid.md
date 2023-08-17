## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Incorrect condition marks valid order as invalid](https://github.com/code-423n4/2022-06-infinity-findings/issues/120) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityOrderBookComplication.sol#L140


# Vulnerability details

## Impact
canExecMatchOrder is having an incorrect check which makes a valid order as invalid. doItemsIntersect function is also checked on sell.nfts, buy.nfts which is incorrect. doItemsIntersect should only be checked in reference to constructedNfts

## Proof of Concept
1. Assume buy has nfts {A,B,C}, sell has nft {A,B}, constructedNfts has nft {A}, buy.constraints[0]/sell.constraints[0]/numConstructedItems is 1

2. Ideally this order should match since constructedNfts {A} is present in both buy and sell

3. But this will not match since doItemsIntersect(sell.nfts, buy.nfts) will fail because of item C which is not present in sell

## Recommended Mitigation Steps
Remove doItemsIntersect(sell.nfts, buy.nfts) from InfinityOrderBookComplication.sol#L140

