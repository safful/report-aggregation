## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [NFTXStakingZap and NFTXMarketplaceZap's transferFromERC721 transfer Cryptokitties to the wrong address](https://github.com/code-423n4/2021-12-nftx-findings/issues/185) 

# Handle

hyh


# Vulnerability details

## Impact

`transferFromERC721(address assetAddr, uint256 tokenId, address to)` should transfer from `msg.sender` to `to`.
It transfers to `address(this)` instead when ERC721 is Cryptokitties.
As there is no additional logic for this case it seems to be a mistake that leads to wrong NFT accounting after such a transfer as NFT will be missed in the vault (which is `to`).

## Proof of Concept

NFTXStakingZap:
transferFromERC721
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L416

NFTXMarketplaceZap:
transferFromERC721
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L556

Both functions are called by user facing Marketplace buy/sell and Staking addLiquidity/provideInventory functions.

## Recommended Mitigation Steps

Fix the address:

Now:
```
// Cryptokitties.
data = abi.encodeWithSignature("transferFrom(address,address,uint256)", msg.sender, address(this), tokenId);
```

To be:
```
// Cryptokitties.
data = abi.encodeWithSignature("transferFrom(address,address,uint256)", msg.sender, to, tokenId);
```


