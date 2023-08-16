## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Local variables shadowing](https://github.com/code-423n4/2021-12-nftx-findings/issues/123) 

# Handle

sirhashalot


# Vulnerability details

## Impact

If two variables with the same name exist in a function, but one is imported from another contract while the other is created locally, it is unclear which value is being used or should be used. Avoiding variable name collisions avoids confusion and the risks of using the wrong variable.

https://swcregistry.io/docs/SWC-119

## Proof of Concept

Several instance of this issue exist.
1. mintAndSell1155() function in NFTXMarketplaceZap.sol has `uint256[] memory amounts` as an input parameter and later it is redeclared in the function
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L368,L376
2. pauseFeeDistribution() function in NFTXSimpleFeeDistributor.sol uses a pause return bool which shadows PausableUpgradeable.pause
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L140
3. transferFromERC721() function in NFTXStakingZap.sol declares an `address owner` which shadows Ownable.owner():
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXStakingZap.sol#L422
4. transferFromERC721() function in NFTXMarketplaceZap.sol declares an `address owner` which shadows Ownable.owner():
https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L562

## Tools Used

Slither, shadowing-local detector: https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing

## Recommended Mitigation Steps

Rename local variables to avoid shadowing. For instance, add an underscore in front of the name of local variables.

