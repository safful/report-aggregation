## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [`Auction:createBid`: can take a bid with the same `highestBid` if (highestBid * minBidIncrement < 100)](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/349) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/auction/Auction.sol#L123


# Vulnerability details

## Impact

A bidder can outbid previous bid with the same value, if the `(previous bid * minBidIncrement < 100)`.

## Proof of Concept

```solidity
// Auction.sol

// createBid
117             unchecked {
118                 // Compute the minimum bid
119                 minBid = highestBid + ((highestBid * settings.minBidIncrement) / 100);
120             }
121
122             // Ensure the incoming bid meets the minimum
123             if (msg.value < minBid) revert MINIMUM_BID_NOT_MET();

331     function setMinimumBidIncrement(uint256 _percentage) external onlyOwner {
332         settings.minBidIncrement = SafeCast.toUint8(_percentage);
333
334         emit MinBidIncrementPercentageUpdated(_percentage);
335     }
```

When the `minBid` (defined in the line 119) is the same as the current `highestBid`, one can call `createBid` with the same value as the current `highestBid` (line 123). It means that the new bidder will get the Bid, even though the previous bidder has the same bid and was earlier.
The `minBid` can be the same as the `highesBid` when `highestBid * minBidIncrement` is less than 100. So either the `highestBid` or `minBinIncrement` is too small, a bidder can overbid the precious one with the same amount of value.

The first bid should be higher or equal to the `reservePrice`. However, there is no safe guard against setting small `reservePrice` and `minBidIncrement`.

For example, let's say the `settings.minBidIncrement` is set to zero. Alice called `createBid` with 1 ether and is the current highestBidder with the `highestBid` of 1 ether. Bob calls `createBid` with 1 ether. The `minBid` in the line 119 will be 1ether as `minBidIncrement` is set to zero. In the line 123 the `msg.value` is 1 ether is the same as `minBid` therefore it will not revert. And now Bob is the `highestBidder` even though he bid the same value after Alice.

## Tools Used

None

## Recommended Mitigation Steps

Revert if the `msg.value` is the same as the `minBid`:
```solidity
// Auction.sol

// createBid
117             unchecked {
118                 // Compute the minimum bid
119                 minBid = highestBid + ((highestBid * settings.minBidIncrement) / 100);
120             }
121
122             // Ensure the incoming bid meets the minimum
- 		if (msg.value < minBid) revert MINIMUM_BID_NOT_MET();
+		if (msg.value <= minBid) revert MINIMUM_BID_NOT_MET();
```

<!-- zzzitron M00 -->

