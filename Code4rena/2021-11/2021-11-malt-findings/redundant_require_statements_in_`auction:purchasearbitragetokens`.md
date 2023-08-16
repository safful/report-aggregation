## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant require statements in `Auction:purchaseArbitrageTokens`](https://github.com/code-423n4/2021-11-malt-findings/issues/108) 

# Handle

loop


# Vulnerability details

When invoking `purchaseArbitrageTokens()` is will first check whether the auction is active using:
```
require(auctionActive(currentAuctionId), "No auction running");
```
`auctionActive()` checks for the following things:
```
auction.active && now >= auction.startingTime;
```
As a result the require statement will fail if either `!auction.active` or `now < auction.startingTime`. 

Later on in `purchaseArbitrageTokens()` two more require statements will check the same thing:
```
require(auction.startingTime <= now, "Auction hasn't started yet");
(...)
 require(auction.active == true, "Auction is not active");
```
These will always pass if `auctionActive(currentAuctionId)` is `true` and never be reached if it is `false`, making them redundant.

## Proof of Concept
- https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L178
- https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L188-L190
- https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L268-L272

## Recommended Mitigation Steps
Remove redundant require statements

