## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Some real-world NFT tokens may support both ERC721 and ERC1155 standards, which may break `InfinityExchange::_transferNFTs`](https://github.com/code-423n4/2022-06-infinity-findings/issues/43) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1062-L1072


# Vulnerability details

## Impact

Many real-world NFT tokens may support both ERC721 and ERC1155 standards, which may break `InfinityExchange::_transferNFTs`, i.e., transferring less tokens than expected.

For example, the asset token of [The Sandbox Game](https://www.sandbox.game/en/), a Top20 ERC1155 token on [Etherscan](https://etherscan.io/tokens-nft1155?sort=7d&order=desc), supports both ERC1155 and ERC721 interfaces. Specifically, any ERC721 token transfer is regarded as an ERC1155 token transfer with only one item transferred ([token address](https://etherscan.io/token/0xa342f5d851e866e18ff98f351f2c6637f4478db5) and [implementation](https://etherscan.io/address/0x7fbf5c9af42a6d146dcc18762f515692cd5f853b#code#F2#L14)).

Assuming there is a user tries to buy two tokens of Sandbox's ASSETs with the same token id, the actual transferring is carried by `InfinityExchange::_transferNFTs` which first checks ERC721 interface supports and then ERC1155.

```solidity=
  function _transferNFTs(
    address from,
    address to,
    OrderTypes.OrderItem calldata item
  ) internal {
    if (IERC165(item.collection).supportsInterface(0x80ac58cd)) {
      _transferERC721s(from, to, item);
    } else if (IERC165(item.collection).supportsInterface(0xd9b67a26)) {
      _transferERC1155s(from, to, item);
    }
  }
```

The code will go into `_transferERC721s` instead of `_transferERC1155s`, since the Sandbox's ASSETs also support ERC721 interface. Then, 

```solidity=
  function _transferERC721s(
    address from,
    address to,
    OrderTypes.OrderItem calldata item
  ) internal {
    uint256 numTokens = item.tokens.length;
    for (uint256 i = 0; i < numTokens; ) {
      IERC721(item.collection).safeTransferFrom(from, to, item.tokens[i].tokenId);
      unchecked {
        ++i;
      }
    }
  }
```

Since the `ERC721(item.collection).safeTransferFrom` is treated as an ERC1155 transferring with one item ([reference](https://etherscan.io/address/0x7fbf5c9af42a6d146dcc18762f515692cd5f853b#code#F2#L833)), there is only one item actually gets traferred.

That means, the user, who barely know the implementation details of his NFTs, will pay the money for two items but just got one.

Note that the situation of combining ERC721 and ERC1155 is prevalent and poses a great vulnerability of the exchange contract.   

## Proof of Concept
Check the return values of [Sandbox's ASSETs](https://etherscan.io/token/0xa342f5d851e866e18ff98f351f2c6637f4478db5)'s `supportInterface`, both `supportInterface(0x80ac58cd)` and `supportInterface(0xd9b67a26)` return true.


## Tools Used
Manual Inspection

## Recommended Mitigation Steps
Reorder the checks,e.g., 

```solidity=
  function _transferNFTs(
    address from,
    address to,
    OrderTypes.OrderItem calldata item
  ) internal {
    if (IERC165(item.collection).supportsInterface(0xd9b67a26)) {
      _transferERC1155s(from, to, item);
    } else if (IERC165(item.collection).supportsInterface(0x80ac58cd)) {
      _transferERC721s(from, to, item);
    }
  }
```


