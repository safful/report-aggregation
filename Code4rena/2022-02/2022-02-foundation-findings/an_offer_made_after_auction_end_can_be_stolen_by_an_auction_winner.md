## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [An offer made after auction end can be stolen by an auction winner](https://github.com/code-423n4/2022-02-foundation-findings/issues/49) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L556-L560


# Vulnerability details


## Impact

An Offer which is made for an NFT when auction has ended, but its winner hasn't received the NFT yet, can be stolen by this winner as `_transferFromEscrow` being called by `_acceptOffer` will transfer the NFT to the winner, finalising the auction, while no transfer to the user who made the offer will happen.

This way the auction winner will obtain both the NFT and the offer amount after the fees at no additional cost, at the expense of the user who made the offer.

## Proof of Concept

When an auction has ended, there is a possibility to make the offers for an auctioned NFT as:

`makeOffer` checks `_isInActiveAuction`:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L200

`_isInActiveAuction` returns false when `auctionIdToAuction[auctionId].endTime < block.timestamp`, so `makeOffer` above can proceed:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L666-L669

Then, the auction winner can call `acceptOffer -> _acceptOffer` (or `setBuyPrice -> _autoAcceptOffer -> _acceptOffer`).

`_acceptOffer` will try to transfer directly, and then calls `_transferFromEscrow`:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L262-L271

If the auction has ended, but a winner hasn't picked up the NFT yet, the direct transfer will fail, proceeding with `_transferFromEscrow` in the FNDNFTMarket defined order:
```
function _transferFromEscrow(
address nftContract,
uint256 tokenId,
address recipient,
address seller
) internal override(NFTMarketCore, NFTMarketReserveAuction, NFTMarketBuyPrice, NFTMarketOffer) {
super._transferFromEscrow(nftContract, tokenId, recipient, seller);
}
```

NFTMarketOffer._transferFromEscrow will call super as `nftContractToIdToOffer` was already deleted:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L296-L302

NFTMarketBuyPrice._transferFromEscrow will call super as there is no buy price set:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketBuyPrice.sol#L283-L293

Finally, NFTMarketReserveAuction._transferFromEscrow will send the NFT to the winner via `_finalizeReserveAuction`, not to the user who made the offer:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L556-L560

The `recipient` user who made the offer is not present in this logic, the NFT is being transferred to the `auction.bidder`, and the original `acceptOffer` will go through successfully.

## Recommended Mitigation Steps

An attempt to set a buy price from auction winner will lead to auction finalisation, so `_buy` cannot be called with a not yet finalised auction, this way the NFTMarketReserveAuction._transferFromEscrow L550-L560 logic is called from the NFTMarketOffer._acceptOffer only:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L270

is the only user of 

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketReserveAuction.sol#L550-L560


This way the fix is to update L556-L560 for the described case as:

Now:
```
// Finalization will revert if the auction has not yet ended.
_finalizeReserveAuction(auctionId, false);

// Finalize includes the transfer, so we are done here.
return;
```

To be, we leave the NFT in the escrow and let L564 super call to transfer it to the recipient:
```
// Finalization will revert if the auction has not yet ended.
_finalizeReserveAuction(auctionId, true);
```

