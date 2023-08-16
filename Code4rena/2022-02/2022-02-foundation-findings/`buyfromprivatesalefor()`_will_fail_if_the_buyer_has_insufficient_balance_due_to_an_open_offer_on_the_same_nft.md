## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`buyFromPrivateSaleFor()` Will Fail if The Buyer Has Insufficient Balance Due to an Open Offer on The Same NFT](https://github.com/code-423n4/2022-02-foundation-findings/issues/77) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketPrivateSale.sol#L143-L150


# Vulnerability details

## Impact

The `buyFromPrivateSaleFor()` function allows sellers to make private sales to users. If insufficient `ETH` is provided to the function call, the protocol will attempt to withdraw the amount difference from the user's unlocked balance. However, if the same user has an open offer on the same NFT, then these funds will remain locked until expiration. As a result, the user cannot make use of these locked funds even though they may be needed for a successful sale.

## Proof of Concept

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider adding a `_cancelBuyersOffer()` call to the `buyFromPrivateSaleFor()` function. This should be added only to the case where insufficient `ETH` was provided to the trade. By cancelling the buyer's offer on the same NFT, we can guarantee that the user has access to the correct amount of funds.

