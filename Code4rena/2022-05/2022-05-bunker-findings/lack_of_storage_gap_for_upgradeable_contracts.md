## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Lack of Storage Gap for Upgradeable Contracts](https://github.com/code-423n4/2022-05-bunker-findings/issues/53) 

# Lines of code

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/ERC1155Enumerable.sol#L70-L71
https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/CNft.sol#L282-L283


# Vulnerability details

## Impact

The code base contains several upgradeable contracts that inherit other upgradeable contracts, including `ERC1155Enumerable.sol` and `CNFT.sol`. These contracts currently do not contain any storage gap.

For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely. 

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable


## Proof of Concept

All OpenZeppelin upgradeable contract templates contain storage gap, including `ReentrancyGuardUpgradeable`, `OwnableUpgradeable` and `ERC1155Upgradeable` that are used in this project. Refer to the bottom of the code in the links below: 

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/security/ReentrancyGuardUpgradeable.sol

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC1155/ERC1155Upgradeable.sol

The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this article: 

https://docs.openzeppelin.com/contracts/3.x/upgradeable

Note that it isn't enough to simply have the OpenZeppelin base contracts contain storage gaps. In this project, the `CNFt` contract inherits the `ERC1155Enumerable` contract, which inherits `ERC1155Upgradeable`. We know the `ERC1155Upgradeable` contract contains a storage gap, so that contract can add additional variables without affecting its child contracts. However, `ERC1155Enumerable` currently does not contain a storage gap, and if in a future upgrade, a new variable is used in the `ERC1155Enumerable` contract, the storage slot of that new variable would overlap with the existing storage slots that are used by `CNFt` and overwrites it, causing unintended and potentially serious consequences including a complete malfunction of the `CNFt` contract. Refer to the bottom of this link for an example and explanation: 

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

## Tools Used

Manual review

## Recommended Mitigation Steps

Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates. 

```solidity
  uint256[50] private __gap;
```

