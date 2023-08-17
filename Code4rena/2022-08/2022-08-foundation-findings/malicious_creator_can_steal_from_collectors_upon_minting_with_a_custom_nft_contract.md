## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Malicious Creator can steal from collectors upon minting with a custom NFT contract](https://github.com/code-423n4/2022-08-foundation-findings/issues/211) 

# Lines of code

https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/nftDropMarket/NFTDropMarketFixedPriceSale.sol#L207


# Vulnerability details

# Malicious Creator can steal from collectors upon minting with a custom NFT contract

In the case of a fixed price sale where `nftContract` is a custom NFT contract that adheres to `INFTDropCollectionMint`, a malicious creator can set a malicious implementation of `INFTDropCollectionMint.mintCountTo()` that would result in collectors calling this function losing funds without receiving the expected amount of NFTs.

## Impact

Medium


## Proof Of Concept

Here is a [foundry test](https://gist.github.com/joestakey/4b13c7ae6029332da6eaf63b9d2a38bd) that shows a fixed price sale with a malicious NFT contract, where a collector pays for 10 NFTs while only receiving one. It can be described as follow:

- A creator creates a malicious `nftContract` with `mintCountTo` minting only one NFT per call, regardless of the value of `count`

- The creator calls `NFTDropMarketFixedPriceSale.createFixedPriceSale()` to create a sale for `nftContract`, with `limit` set to `15`.

- Bob is monitoring the `CreateFixedPriceSale` event. Upon noticing `CreateFixedPriceSale(customERC721, Alice, price, limit)`, he calls `NFTDropMarketFixedPriceSale.mintFromFixedPriceSale(customERC721, count == 10,)`. He pays the price of `count = 10` NFTs, but because of the logic in `mintCountTo`, only receives one NFT.

Note that `mintCountTo` can be implemented in many malicious ways, this is only one example. Another implementation could simply return `firstTokenId` without performing any minting.

## Tools Used

Manual Analysis, Foundry

## Mitigation

The problem here lies in the implementation of `INFTDropCollectionMint(nftContract).mintCountTo()`. You could add an additional check in `NFTDropMarketFixedPriceSale.mintCountTo()` using `ERC721(nftContract).balanceOf()`. 

```diff
+ uint256 balanceBefore = IERC721(nftContract).balanceOf(msg.sender);
207:     firstTokenId = INFTDropCollectionMint(nftContract).mintCountTo(count, msg.sender);
+ uint256 balanceAfter = IERC721(nftContract).balanceOf(msg.sender);
+ require(balanceAfter == balanceBefore + count, "minting failed")
```
