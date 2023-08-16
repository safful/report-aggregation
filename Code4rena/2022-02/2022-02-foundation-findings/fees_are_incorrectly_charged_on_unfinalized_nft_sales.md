## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fees Are Incorrectly Charged on Unfinalized NFT Sales](https://github.com/code-423n4/2022-02-foundation-findings/issues/73) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L255-L271
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L557
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L510-L515
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketFees.sol#L188-L189


# Vulnerability details

## Impact

Once an auction has ended, the highest bidder now has sole rights to the underlying NFT. By finalizing the auction, fees are charged on the sale and the NFT is transferred to `auction.bidder`. However, if `auction.bidder` accepts an offer before finalization, fees will be charged on the `auction.bidder` sale before the original sale. As a result, it is possible to avoid paying the primary foundation fee as a creator if the NFT is sold by `auction.bidder` before finalization.

## Proof of Concept

Consider the following scenario:
- Alice creates an auction and is the NFT creator.
- Bob bids on the auction and is the highest bidder.
- The auction ends but Alice leaves it in an unfinalized state.
- Carol makes an offer on the NFT which Bob accepts.
- `_acceptOffer()` will distribute funds on the sale between Bob and Carol before distributing funds on the sale between Alice and Bob.
- The first call to `_distributeFunds()` will set the `_nftContractToTokenIdToFirstSaleCompleted` to true, meaning that future sales will only be charged the secondary foundation fee.

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Ensure the `_nftContractToTokenIdToFirstSaleCompleted` is correctly tracked. It might be useful to ensure the distribution of funds are in the order of when the trades occurred. For example, an unfinalized auction should always have its fees paid before other sales.

