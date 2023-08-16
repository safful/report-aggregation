## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No minimum rate in the auction may break the protocol under network failure](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/42) 

# Handle

jonah1005


# Vulnerability details

## Impact
The aution contract decides a new `ibRatio` in the function `settleAuction`. [Auction.sol#L89-L91](https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L89-L91)

```solidity
        uint256 a = factory.auctionMultiplier() * basket.ibRatio();
        uint256 b = (bondTimestamp - auctionStart) * BASE / factory.auctionDecrement();
        uint256 newRatio = a - b;
```

There's a chance that `newRatio` would be really close to zero. This imposes too much risk on the protocol. The network may not really be healthy all the time. Solana and Arbitrum were down and Ethereum was suffered a forking issue recently. Also, the network may be jammed from time to time. This could cause huge damage to a protocol. Please refer to [Black Thursday for makerdao 8.32 million was liquidated for 0 dai](https://medium.com/@whiterabbit_hq/black-thursday-for-makerdao-8-32-million-was-liquidated-for-0-dai-36b83cac56b6)

Given the chance that all user may lose their money, I consider this is a medium-risk issue.


## Proof of Concept
[Black Thursfay for makerdao 8.32 million was liquidated for 0 dai](https://medium.com/@whiterabbit_hq/black-thursday-for-makerdao-8-32-million-was-liquidated-for-0-dai-36b83cac56b6)
[bug-impacting-over-50-of-ethereum-clients-leads-to-fork](https://www.theblockcrypto.com/post/115822/bug-impacting-over-50-of-ethereum-clients-leads-to-fork)

## Tools Used
None

## Recommended Mitigation Steps

I recommend setting a minimum `ibRatio` when a publisher publishes a new index. The auction should be killed if the `ibRatio` is too low.

