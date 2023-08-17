## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [USDT is not supported because of approval mechanism](https://github.com/code-423n4/2022-05-rubicon-findings/issues/100) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathHouse.sol#L180
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L157
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathToken.sol#L256


# Vulnerability details

When using the approval mechanism in USDT, the approval must be set to 0 before it is updated.
In Rubicon, when creating a pair, the paired asset's approval is not set to 0 before it is updated.

## Impact
Can't create pairs with USDT, the most popular stablecoin, as as the approval will revert.

## Proof of Concept
[USDT](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code) reverts on approval if previous allowance is not 0:
```
require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));
```
When creating a pair, Rubicon [approves](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathHouse.sol#L180) the paired asset without first setting it to 0:
```
desiredPairedAsset.approve(pairedPool, initialLiquidityExistingBathToken);
```
Therefore, if desiredPairedAsset is USDT, the function will revert, and pairs with USDT can not be created.

This problem will also manifest in RubiconMarket's [approval function](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L157) and BathToken's [approval function](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/rubiconPools/BathToken.sol#L256), 

## Recommended Mitigation Steps
Set the allowance to 0 before setting it to the new value.

