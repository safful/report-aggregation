## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [AuctionCrowdfund: If the contract was bid on before the NFT was gifted to the contract, lastBid will not be totalContributions](https://github.com/code-423n4/2022-09-party-findings/issues/147) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/AuctionCrowdfund.sol#L233-L242


# Vulnerability details

## Impact
In the finalize function of the AuctionCrowdfund contract, when the contract gets NFT and lastBid_ == 0, it is considered that NFT is gifted to the contract and everyone who contributed wins.
```
        if (nftContract.safeOwnerOf(nftTokenId) == address(this)) {
            if (lastBid_ == 0) {
                // The NFT was gifted to us. Everyone who contributed wins.
                lastBid_ = totalContributions;
```
But if the contract was bid before the NFT was gifted to the contract, then since lastBid_ ! = 0, only the user who contributed at the beginning will win.
## Proof of Concept
https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/AuctionCrowdfund.sol#L233-L242
https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/AuctionCrowdfund.sol#L149-L175
## Tools Used
None
## Recommended Mitigation Steps
Whether or not NFT is free to get should be determined using whether the contract balance is greater than totalContributions