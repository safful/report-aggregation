## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Use safeTransferFrom instead of transferFrom for ERC721 transfers](https://github.com/code-423n4/2022-05-cally-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L199
https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L295
https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L344


# Vulnerability details

## Details & Impact

The `transferFrom()` method is used instead of `safeTransferFrom()`, presumably to save gas. I however argue that this isn’t recommended because:

- [OpenZeppelin’s documentation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-) discourages the use of `transferFrom()`, use `safeTransferFrom()` whenever possible
- Given that any NFT can be used for the call option, there are a few NFTs (here’s an [example](https://github.com/sz-piotr/eth-card-game/blob/master/src/ethereum/contracts/ERC721Market.sol#L20-L31)) that have logic in the `onERC721Received()` function, which is only triggered in the `safeTransferFrom()` function and not in `transferFrom()`

## Recommended Mitigation Steps

Call the `safeTransferFrom()` method instead of `transferFrom()` for NFT transfers. Note that the `CallyNft` contract should inherit the `ERC721TokenReceiver` contract as a consequence.

```solidity
abstract contract CallyNft is ERC721("Cally", "CALL"), ERC721TokenReceiver {...}
```

