## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [NFT Hub implementation deviates from ERC721 for transfer functions](https://github.com/code-423n4/2021-06-realitycards-findings/issues/118) 

# Handle

0xRajeev


# Vulnerability details

## Impact

ERC721 standard and implementation allows the use of approved addresses to affect transfers besides the token owners. However, the L2 NFT Hub implementation deviates from ERC721 by ignoring the presence of any approvers in the overriding function implementations of transferFrom() and safeTransferFrom(). 

Impact: The system interactions with NFT platforms may not work if they expect ERC721 adherence. Users who interact via approved addresses will see their transfers failing for their approved addresses. 

Given that the key value proposition of this project is the use of NFTs, the expectation will be that it is fully compatible with ERC721.


## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/nfthubs/RCNftHubL2.sol#L212-L234

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/00128bd26061986d10172573ceec914a4f3b4d3c/contracts/token/ERC721/ERC721.sol#L158


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add support for approval in NFT transfers.

