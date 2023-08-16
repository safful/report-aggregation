## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [A vault can be locked from MarketplaceZap and StakingZap](https://github.com/code-423n4/2021-12-nftx-findings/issues/107) 

# Handle

p4st13r4


# Vulnerability details

## Impact

Any user that owns a vToken of a particular vault can lock the functionalities of `NFTXMarketplaceZap.sol` and `NFTXStakingZap.sol` for everyone.

Every operation performed by the marketplace, that deals with vToken minting, performs this check:

```jsx
require(balance == IERC20Upgradeable(vault).balanceOf(address(this)), "Did not receive expected balance");
```

A malicious user could transfer any amount > 0 of a vault’vToken to the marketplace (or staking) zap contracts, thus making the vault functionality unavailable for every user on the marketplace

## Proof of Concept

[https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L421](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L421)

[https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L421](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXMarketplaceZap.sol#L421)

## Tools Used

Editor

## Recommended Mitigation Steps

Remove this logic from the marketplace and staking zap contracts, and add it to the vaults (if necessary)

