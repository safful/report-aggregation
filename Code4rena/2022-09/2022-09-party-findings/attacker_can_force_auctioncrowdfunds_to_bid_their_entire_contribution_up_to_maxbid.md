## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Attacker can force AuctionCrowdfunds to bid their entire contribution up to maxBid](https://github.com/code-423n4/2022-09-party-findings/issues/220) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/AuctionCrowdfund.sol#L166-L167


# Vulnerability details

## Description
AuctionCrowdfund's bid() allows any user to compete on an auction on the party's behalf. The code in bid()  forbids placing a bid if party is already winning the auction:
```
if (market.getCurrentHighestBidder(auctionId_) == address(this)) {
            revert AlreadyHighestBidderError();
        }
```
However, it does not account for attackers placing bids from their own wallet, and then immediately overbidding them using the party's funds. This can be used in two ways:
1. Attacker which lists an NFT, can force the party to spend all its funds up to maxBid on the auction, even if the party could have purchased the NFT for much less.
2. Attackers can grief random auctions, making them pay the absolute maximum for the item. Attackers can use this to drive the prices of NFT items up, profiting from this using secondary markets.

## Impact
Parties can be stopped from buying items at a good value without any risk to the attacker.

## Proof of Concept
1. Attacker places an NFT for sale, valued at X
2. Attacker creates an AuctionCrowdfund, with maxBid = Y such that Y = 2X
3. Current bid for the NFT is X - AUCTION_STEP
3. Users contribute to the fund, which now has 1.5X
4. Users call bid() to bid X  for the NFT
5. Attacker bids for the item externally for 1.5X - AUCTION_STEP
6. Attacker calls bid() to bid 1.5X for the NFT
7. Attacker sells his NFT for 1.5X although no one apart from the party was interested in buying it above price X

## Tools Used
Manual audit.

## Recommended Mitigation Steps
Introduce a new option variable to AuctionCrowdfunds called speedBump. Inside the bid() function, calculate seconds since last bid, multiplied by the price change factor. This product must be smaller than the chosen speedBump. Using this scheme, the protocol would have resistance to sudden bid spikes. Optionally, allow a majority funder to override the speed bump.

