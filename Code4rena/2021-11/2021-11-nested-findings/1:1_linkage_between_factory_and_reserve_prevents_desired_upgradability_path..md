## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [1:1 linkage between factory and reserve prevents desired upgradability path.](https://github.com/code-423n4/2021-11-nested-findings/issues/32) 

# Handle

TomFrench


# Vulnerability details

## Impact

NFTs can't be managed from future versions of `NestedFactory` without manual migration or removing support for the previous `NestedFactory`.

## Proof of Concept

From discussion with NestedFinance team members, it's desired that multiple `NestedFactories` can interact with the NFT portfolios and be interoperable into the future.

NestedFinance has two singleton contracts which store the state of NFTs `NestedAsset` and `NestedRecords`

`NestedAsset` allows multiple factories to interact with a given NFT ([asset](https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedAsset.sol#L18-L19))
`NestedRecords` lists a single reserve which holds an NFT's assets ([records](https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedRecords.sol#L32))

This is fine however `NestedFactory` and `NestedReserve` are linked together 1:1 ([factory](https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedFactory.sol#L31), [reserve](https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedReserve.sol#L14))

This means that each NFT can only be managed by a single factory as calls from other factories to the relevant reserve will revert due to insufficient permissions. Should I want to update to use the newest factory I would have to manually migrate my portfolio across.

## Recommended Mitigation Steps

Allow `NestedReserve` to have multiple factories connect to it. Make sure to have the `NestedReserve` secure from reentrancy attacks utilising multiple factories in parallel. 

