## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Dutch auction can be manipulated](https://github.com/code-423n4/2021-11-malt-findings/issues/375) 

# Handle

gzeon


# Vulnerability details

## Impact
When malt is under-peg and the swing trader module do not have enough capital to buy back to peg, a Dutch auction is triggered to sell arb token. The price of the Dutch auction decrease linearly toward endprice until _endAuction() is called. https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L589

_endAuction() is called in 

1. When auction.commitments >= auction.maxCommitments
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L212

2. On stabilize() -> checkAuctionFinalization() -> _checkAuctionFinalization()
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L146
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L754

3. On stabilize() ->_startAuction() -> triggerAuction() -> _checkAuctionFinalization()
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L170
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L754

It is possible manipulate the dutch auction by preventing _endAuction() being called.

## Proof of Concept
Consider someone call purchaseArbitrageTokens with auction.maxCommitments minus 1 wei, `_endAuction` won't be called because auction.commitments < auction.maxCommitments. Further purchase would revert because `purchaseAndBurn` (https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L184) would likely revert since swapping 1 wei in most AMM will fail due to rounding error. Even if it does not revert, there is no incentive to waste gas to purchase 1 wei of token.

As such, the only way for the auction to finalize is to call stabilize(). 
However, this is not immediately possible because it require 
`block.timestamp >= stabilizeWindowEnd` where
`stabilizeWindowEnd = block.timestamp + stabilizeBackoffPeriod`
stabilizeBackoffPeriod is initially set to 5 minutes in the contract

After 5 minute, stabilize() can be called by anyone. By using this exploit, an attacker can guarantee he can purchase at (startingPrice+endingPrice)/2 or lower, given the default 10 minute auctionLength and 5 minute stabilizeBackoffPeriod. (unless a privileged user call stabilize() which override the stability window)

Also note that stabilize() might not be called since there is no incentive.

## Recommended Mitigation Steps
1. Incentivize stabilize() or incentivize a permission-less call to _endAuction()
2. Lock-in auction price when user commit purchase

