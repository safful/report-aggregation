## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [NFTXVaultFactoryUpgradeable implementation can be replaced in production breaking the system](https://github.com/code-423n4/2021-12-nftx-findings/issues/177) 

# Handle

hyh


# Vulnerability details

## Impact

`NFTXVaultFactory` contract holds information regarding vaults, assets and permissions (vaults, _vaultsForAsset and excludedFromFees mappings).
As there is no mechanics present that transfers this information to another implementation, the switch of nftxVaultFactory to another address performed while in production will break the system.

## Proof of Concept

`setNFTXVaultFactory` function allows an owner to reset `nftxVaultFactory` without restrictions in the following contracts:

NFTXLPStaking
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol#L59

NFTXInventoryStaking
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXInventoryStaking.sol#L51

NFTXSimpleFeeDistributor
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L135

## Recommended Mitigation Steps

Either restrict the ability to change the factory implementation to pre-production stages or make `nftxVaultFactory` immutable by allowing changing it only once:

Now:
```
function setNFTXVaultFactory(address newFactory) external virtual override onlyOwner {
		require(newFactory != address(0));
		nftxVaultFactory = INFTXVaultFactory(newFactory);
}
```

To be:
```
function setNFTXVaultFactory(address newFactory) external virtual override onlyOwner {
		require(nftxVaultFactory == address(0), "nftxVaultFactory is immutable");
		nftxVaultFactory = INFTXVaultFactory(newFactory);
}
```

If the implementation upgrades in production is desired, the factory data migration logic should be implemented and then used atomically together with the implementation switch in all affected contracts.


