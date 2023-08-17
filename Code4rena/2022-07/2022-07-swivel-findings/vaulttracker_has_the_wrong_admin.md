## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [VaultTracker has the wrong admin](https://github.com/code-423n4/2022-07-swivel-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/main/Marketplace/MarketPlace.sol#L77
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/Creator.sol#L41
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L32


# Vulnerability details

## Description

`MarketPlace.createMarket()` calls `Creator.create()` which creates an instance of `ZcToken` and a `VaultTracker`. `VaultTracker` takes `msg.sender` as the admin. We know that if contract A calls contract B which calls contract C, `msg.sender` in contract C is the address of B i.e. the `msg.sender` in VaultTracker is the address of the creator contract. However, the creator contract is not able (and not supposed to) interact with the VaultTracker unlike the marketplace contract.

## Tools used
Manual analysis

## Recommended Mitigation Steps

Modify the constructor of the VaultTracker contract so that the creator contract can pass in msg.sender (MarketPlace’s address) to be used as admin.

