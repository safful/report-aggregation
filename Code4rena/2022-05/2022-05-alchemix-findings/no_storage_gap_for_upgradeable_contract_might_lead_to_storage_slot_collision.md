## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No Storage Gap for Upgradeable Contract Might Lead to Storage Slot Collision](https://github.com/code-423n4/2022-05-alchemix-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/AlchemicTokenV2Base.sol#L20
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/CrossChainCanonicalBase.sol#L12
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/TransmuterV2.sol#L26
https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/CrossChainCanonicalAlchemicTokenV2.sol#L7


# Vulnerability details

## Impact

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable


## Proof of Concept

Several contracts are intended to be upgradeable contracts in the code base, including 
- AlchemicTokenV2Base
- CrossChainCanonicalBase
- CrossChainCanonicalAlchemicTokenV2
- TransmuterV2

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this article:

https://docs.openzeppelin.com/contracts/3.x/upgradeable

As an example, both the `AlchemicTokenV2Base` and the `CrossChainCanonicalBase` are intended to act as the base contracts in the project. If the contract inheriting the base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability. 


## Tools Used

Manual review

## Recommended Mitigation Steps

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.

```solidity
uint256[50] private __gap;
```

