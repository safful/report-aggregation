## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [NFT Owner can stuck Crowdfund user funds](https://github.com/code-423n4/2022-09-party-findings/issues/197) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/AuctionCrowdfund.sol#L236


# Vulnerability details

## Impact
Consider a scenario where few users contributed in auction but noone has placed any bid due to reason like NFT price crash etc. So there was 0 bid, nft owner could seize the crowdfund users fund until they pay a ransom amount as shown below.

## Proof of Concept
1. NFT N auction is going on
2. CrowdFund users have contributed 100 amount for this auction
3. Bidding has not been done yet
4. A news came for this NFT owner which leads to crashing of this NFT price
5. CrowdFund users are happy that they have not bided and are just waiting for auction to complete so that they can get there refund
6. NFT owner realizing this blackmails the CrowdFund users to send him amount 50 or else he would send this worthless NFT to the Crowdfund Auction contract basically stucking all crowdfund users fund. CrowdFund users ignore this and simply wait for auction to end
7. Once auction completes [finalize function](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/crowdfund/AuctionCrowdfund.sol#L196) is called

```
function finalize(FixedGovernanceOpts memory governanceOpts)
        external
        onlyDelegateCall
        returns (Party party_)
    {
...
 if (nftContract.safeOwnerOf(nftTokenId) == address(this)) {
            if (lastBid_ == 0) {
                // The NFT was gifted to us. Everyone who contributed wins.
                lastBid_ = totalContributions;
                if (lastBid_ == 0) {
                    // Nobody ever contributed. The NFT is effectively burned.
                    revert NoContributionsError();
                }
                lastBid = lastBid_;
            }
            // Create a governance party around the NFT.
            party_ = _createParty(
                _getPartyFactory(),
                governanceOpts,
                nftContract,
                nftTokenId
            );
            emit Won(lastBid_, party_);
        } 
...
}
```

8. Before calling finalize the lastBid was 0 since no one has bid on this auction but lets see what happens on calling finalize

9. Since NFT owner has transferred NFT to this contract so below statement holds true and lastBid_ is also 0 since no one has bided

```
if (lastBid_ == 0) {
                lastBid_ = totalContributions;
```

10. This means now lastBid_ is changed to totalContributions which is 100 so crowdfund users funds will not be refunded and they will end up with non needed NFT. 

## Recommended Mitigation Steps
Remove the line lastBid_ = totalContributions; and let it be the last bid amount which crowdfund users actually bided with.