## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [NFT of NFT collection or NFT drop collection can be locked when calling _mint or mintCountTo function to mint it to a contract that does not support ERC721 protocol](https://github.com/code-423n4/2022-08-foundation-findings/issues/183) 

# Lines of code

https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTCollection.sol#L262-L274
https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTDropCollection.sol#L171-L187


# Vulnerability details

## Impact
When calling the following `_mint` or `mintCountTo` function for minting an NFT of a NFT collection or NFT drop collection, the OpenZeppelin's `ERC721Upgradeable` contract's [`_mint`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/ERC721Upgradeable.sol#L284-L296) function is used to mint the NFT to a receiver. If such receiver is a contract that does not support the ERC721 protocol, the NFT will be locked and cannot be retrieved.

https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTCollection.sol#L262-L274
```
  function _mint(string calldata tokenCID) private onlyCreator returns (uint256 tokenId) {
    require(bytes(tokenCID).length != 0, "NFTCollection: tokenCID is required");
    require(!cidToMinted[tokenCID], "NFTCollection: NFT was already minted");
    unchecked {
      // Number of tokens cannot overflow 256 bits.
      tokenId = ++latestTokenId;
      require(maxTokenId == 0 || tokenId <= maxTokenId, "NFTCollection: Max token count has already been minted");
      cidToMinted[tokenCID] = true;
      _tokenCIDs[tokenId] = tokenCID;
      _mint(msg.sender, tokenId);
      emit Minted(msg.sender, tokenId, tokenCID, tokenCID);
    }
  }
```

https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTDropCollection.sol#L171-L187
```
  function mintCountTo(uint16 count, address to) external onlyMinterOrAdmin returns (uint256 firstTokenId) {
    require(count != 0, "NFTDropCollection: `count` must be greater than 0");

    unchecked {
      // If +1 overflows then +count would also overflow, unless count==0 in which case the loop would exceed gas limits
      firstTokenId = latestTokenId + 1;
    }
    latestTokenId = latestTokenId + count;
    require(latestTokenId <= maxTokenId, "NFTDropCollection: Exceeds max tokenId");

    for (uint256 i = firstTokenId; i <= latestTokenId; ) {
      _mint(to, i);
      unchecked {
        ++i;
      }
    }
  }
```

For reference, [OpenZeppelin's documentation for `_mint`](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_mint-address-uint256-) states: "Usage of this method is discouraged, use _safeMint whenever possible".

## Proof of Concept
The following steps can occur when minting an NFT of a NFT collection or NFT drop collection.
1. The [`_mint`](https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTCollection.sol#L262-L274) or [`mintCountTo`](https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTDropCollection.sol#L171-L187) function is called with `msg.sender` or the `to` input corresponding to a contract.
2. The OpenZeppelin's `ERC721Upgradeable` contract's [`_mint`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/ERC721Upgradeable.sol#L284-L296) function is called with `msg.sender` or `to` used in Step 1 as the receiver address.
3. Since calling the OpenZeppelin's `ERC721Upgradeable` contract's [`_mint`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/ERC721Upgradeable.sol#L284-L296) function does not execute the same contract's [`_checkOnERC721Received`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC721/ERC721Upgradeable.sol#L400-L422) function, it is unknown if the receiving contract inherits from the `IERC721ReceiverUpgradeable` interface and implements the `onERC721Received` function or not. It is possible that the receiving contract does not support the ERC721 protocol, which causes the minted NFT to be locked.

## Tools Used
VSCode

## Recommended Mitigation Steps
https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTCollection.sol#L271 can be changed to the following code.
```
_safeMint(msg.sender, tokenId);
```

Also, https://github.com/code-423n4/2022-08-foundation/blob/main/contracts/NFTDropCollection.sol#L182 can be changed to the following code.
```
_safeMint(to, i);
```