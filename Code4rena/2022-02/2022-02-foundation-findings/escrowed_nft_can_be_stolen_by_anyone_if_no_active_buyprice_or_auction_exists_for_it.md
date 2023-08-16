## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Escrowed NFT can be stolen by anyone if no active buyPrice or auction exists for it](https://github.com/code-423n4/2022-02-foundation-findings/issues/51) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketCore.sol#L77-L87


# Vulnerability details

	

## Impact

If a NFT happens to be in escrow with neither buyPrice, nor auction being initialised for it, there is a way to obtain it for free by any actor via `makeOffer`, `acceptOffer` combination.

I.e. a malicious user can track the FNDNFTMarket contract and obtain any NFT from it for which there are no buyPrice or auction structures initialised. For example, if a NFT is mistakenly sent to the contract, an attacker can immediately steal it.

This will happen as NFT is being guarded by buyPrice and auction structures only. The severity here is medium as normal usage of the system imply that either one of them is initialised (NFT was sent to escrow as a part of `setBuyPrice` or `createReserveAuction`, and so one of the structures is present), so this seems to leave only mistakenly sent assets exposed.

## Proof of Concept

An attacker can make a tiny offer with `makeOffer`:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L189

Then call `acceptOffer`, which will lead to `_acceptOffer`.

Direct NFT transfer will fail in `_acceptOffer` as the NFT is being held by the contract and `_transferFromEscrow` will be called instead:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketOffer.sol#L262-L271

`_transferFromEscrow` calls will proceed according to the FNDNFTMarket defined order:
```
function _transferFromEscrow(
...
) internal override(NFTMarketCore, NFTMarketReserveAuction, NFTMarketBuyPrice, NFTMarketOffer) {
   super._transferFromEscrow(nftContract, tokenId, recipient, seller);
}
```

If there are no corresponding structures, the NFTMarketOffer, NFTMarketBuyPrice and NFTMarketReserveAuction versions of `_transferFromEscrow` will pass through the call to NFTMarketCore's plain transfer implementation:

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketCore.sol#L77-L87

This will effectively transfer the NFT to the attacker, which will pay gas costs and an arbitrary small offer price for it.

## Recommended Mitigation Steps

Consider adding additional checks to control who can obtain unallocated NFTs from the contract.

Protocol controlled entity can handle such cases manually by initial sender's request.

