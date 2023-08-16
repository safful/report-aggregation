## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Auction.sol#settleAuction()` late auction bond could potentially not being able to be settled, cause funds loss to bonder](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/90) 

# Handle

WatchPug


# Vulnerability details

The `newRatio` that determines `tokensNeeded` to settle the auction is calculated based on `auctionMultiplier`, `bondTimestamp - auctionStart` and `auctionDecrement`.

```solidity=
uint256 a = factory.auctionMultiplier() * basket.ibRatio();
uint256 b = (bondTimestamp - auctionStart) * BASE / factory.auctionDecrement();
uint256 newRatio = a - b;
```

However, if an auction is bonded late (`bondTimestamp - auctionStart` is a large number), and/or the `auctionMultiplier` is small enough, and/or the `auctionDecrement` is small enough, that makes `b` to be greater than `a`, so that `uint256 newRatio = a - b;` will revert on underflow.

This might seem to be an edge case issue, but considering that a rebalance auction of a bag of shitcoin to high-value tokens might just end up being bonded at the last minute, with a `newRatio` near zero. When we take the time between the bonder submits the transaction and it got packed into a block, it's quite possible that the final `bondTimestamp` gets large enough to revet `a - b`.

### Impact

An auction successfully bonded by a regular user won't be able to be settled, and the user will lose the bond.

### Proof of Concept

With the configuration of:

basket.ibRatio = 1e18
factory.auctionDecrement = 5760 (Blocks per day)
factory.auctionMultiplier = 2

1. Create an auction;
2. The auction remain inactive (not get bonded) for more than 2 days (>11,520 blocks);
3. Call `bondForRebalance()` and it will succeed;
4. Calling `settleAuction()` will always revert.

### Recommended Mitigation Steps

Calculate and require `newRatio > 0` in `bondForRebalance()`, or limit the max value of decrement and make sure newRatio always > 0 in `settleAuction()`.

