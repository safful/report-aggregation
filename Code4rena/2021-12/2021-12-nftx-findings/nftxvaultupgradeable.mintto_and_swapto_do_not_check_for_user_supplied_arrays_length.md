## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [NFTXVaultUpgradeable.mintTo and swapTo do not check for user supplied arrays length](https://github.com/code-423n4/2021-12-nftx-findings/issues/111) 

# Handle

hyh


# Vulnerability details

# Impact

On calling with arrays of different lengths various malfunctions are possible as the arrays are used as given.
mintTo and swapTo outcome will not be as expected by a caller.

## Proof of Concept

The arrays are used whenever Vault is ERC1155, i.e. when is1155 is true.

swap -> swapTo uses the arrays as given:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXVaultUpgradeable.sol#L258

mint -> mintTo, arrays are passed on without checks to receiveNFTs:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXVaultUpgradeable.sol#L190

receiveNFTs uses the arrays in a loop, assuming equal lengths without a check:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXVaultUpgradeable.sol#L389

## Recommended Mitigation Steps

Add `require(tokenIds.length == amounts.length, "tokenIds and amounts length should match")` check in the beginning of public mintTo and swapTo endpoints.


