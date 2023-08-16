## Tags

- bug
- 0 (Non-critical)
- sponsor acknowledged
- sponsor confirmed

# [Internal functions names should start with underscore](https://github.com/code-423n4/2021-12-nftx-findings/issues/124) 

# Handle

sirhashalot


# Vulnerability details

## Impact

Multiple internal functions do not have a name that starts with an underscore. The lack of clarity over the functions visibility could lead to misuse of these functions.

## Proof of Concept
Both the NFTXMarketplaceZap.sol and NFTXStakingZap.sol contracts have three internal functions names without underscores:
approveERC721 in NFTXMarketplaceZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L574
approveERC721 in NFTXStakingZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L434
pairFor in NFTXMarketplaceZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L593
pairFor in NFTXStakingZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L453
sortTokens in NFTXMarketplaceZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L604
sortTokens in NFTXStakingZap.sol: https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L464

## Tools Used

Manual review

## Recommended Mitigation Steps

Rename internal functions following best practices to clarify function visibility

