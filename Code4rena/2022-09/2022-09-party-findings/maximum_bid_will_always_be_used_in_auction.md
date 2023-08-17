## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- edited-by-warden

# [Maximum bid will always be used in Auction](https://github.com/code-423n4/2022-09-party-findings/issues/179) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/AuctionCrowdfund.sol#L149


# Vulnerability details

## Impact
AuctionCrowdfund contract is designed in a way to allow bidding max upto maximumBid. But due to a flaw, anyone (including NFT seller) can make sure that CrowdFund bid always remain equal to maximumBid thus removing the purpose of maximumBid. This also causes loss to Party participating in this Auction as the auction will always end up with maximumBid even when it could have stopped with lower bid as shown in POC

## Proof of Concept
1. An auction is started for NFT N in the market
2. Party Users P1 starts an AuctionCrowdfund with maximumBid as 100 for this auction.

```
function initialize(AuctionCrowdfundOptions memory opts)
        external
        payable
        onlyConstructor
    {
...
maximumBid = opts.maximumBid;
...
}
```

3. P1 bids amount 10 for the NFT N using [bid function](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/AuctionCrowdfund.sol#L149)
4. Some bad news arrives for the NFT collection including NFT N reducing its price
5. P1 decides not to bid more than amount 10 due to this news
6. NFT collection owner who is watching this AuctionCrowdfund observes that bidding is only 10 but Party users have maximumBid of 100
7. NFT collection owner  asks his friend to bid on this NFT in the auction market (different from crowd fund)
8. NFT collection owner now takes advantage of same and himself calls the bid function of AuctionCrowdfund via Proxy

```
function bid() external onlyDelegateCall {
...
}
```

9. Now since last bid belongs to collection owner friend, so AuctionCrowdfund contract simply extends its bid further 

```
if (market.getCurrentHighestBidder(auctionId_) == address(this)) {
            revert AlreadyHighestBidderError();
        }
        // Get the minimum necessary bid to be the highest bidder.
        uint96 bidAmount = market.getMinimumBid(auctionId_).safeCastUint256ToUint96();
        // Make sure the bid is less than the maximum bid.
        if (bidAmount > maximumBid) {
            revert ExceedsMaximumBidError(bidAmount, maximumBid);
        }
        lastBid = bidAmount;
```

10. NFT collection owner keeps repeating step 7-9 until AuctionCrowdfund reaches the final maximum bid of 100
11. After auction completes, collection owner gets 100 amount instead of 10 even though crowd fund users never bidded for amount 100

## Recommended Mitigation Steps
maximumbid concept can easily be bypassed as shown above and will not make sense. Either remove it completely
   OR
bid function should only be callable via crowdfund members then attacker would be afraid if new bid will come or not and there should be a consensus between crowdfund members before bidding which will protect this scenario