## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [The `FERC1155.sol` don't respect the EIP2981](https://github.com/code-423n4/2022-07-fractional-findings/issues/544) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/main/src/FERC1155.sol#L31-L34


# Vulnerability details

## Impact

The [EIP-2981: NFT Royalty Standard](https://eips.ethereum.org/EIPS/eip-2981) implementation is incomplete, missing the implementation of `function supportsInterface(bytes4 interfaceID) external view returns (bool);` from the [EIP-165: Standard Interface Detection](https://eips.ethereum.org/EIPS/eip-165)

## Proof of Concept

A marketplace implemented royalties could check if the NFT have royalties, but if don't add the interface of `ERC2981` on the `_registerInterface`, the marketplace can't know if this NFT haves

## Tools Used

Manual Review

## Recommended Mitigation Steps

Like in [solmate ERC1155.sol](https://github.com/Rari-Capital/solmate/blob/03e425421b24c4f75e4a3209b019b367847b7708/src/tokens/ERC1155.sol#L137-L146) add the `ERC2981` interfaceId on the `FERC1155` contract
```solidity
    /*//////////////////////////////////////////////////////////////
                              ERC165 LOGIC
    //////////////////////////////////////////////////////////////*/

    function supportsInterface(bytes4 interfaceId) public view  override returns (bool) {
        return
            super.supportsInterface(interfaceId) ||
            interfaceId == 0x2a55205a; // ERC165 Interface ID for ERC2981
    }
```

