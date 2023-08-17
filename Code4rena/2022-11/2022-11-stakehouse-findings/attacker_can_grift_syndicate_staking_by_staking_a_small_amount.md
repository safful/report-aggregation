## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-12

# [Attacker can grift syndicate staking by staking a small amount](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/146) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/a0558ed7b12e1ace1fe5c07970c7fc07eb00eebd/contracts/liquid-staking/LiquidStakingManager.sol#L882
https://github.com/code-423n4/2022-11-stakehouse/blob/23c3cf65975cada7fd2255a141b359a6b31c2f9c/contracts/syndicate/Syndicate.sol#L22


# Vulnerability details

## Impact
`LiquidStakingManager._autoStakeWithSyndicate` always stakes a fixed amount of 12 ETH. However, `Syndicate.stake` only allows a total staking amount of 12 ETH and reverts otherwise:
```solidity
if (_sETHAmount + totalStaked > 12 ether) revert InvalidStakeAmount();
```
An attacker can abuse this and front-run calls to `mintDerivatives` (which call `_autoStakeWithSyndicate` internally). Because `Syndicate.stake` can be called by everyone, he can stake the minimum amount (1 gwei) such that the `mintDerivatives` call fails.

## Proof Of Concept
As soon as there is a `mintDerivatives` call in the mempool, an attacker (that owns sETH) calls `Syndicate.stake` with an amount of 1 gwei. `_autoStakeWithSyndicate` will still call `Syndicate.stake` with 12 ether. However, `_sETHAmount + totalStaked > 12 ether` will then be true, meaning that the call will revert.

## Recommended Mitigation Steps
Only allow staking through the LiquidStakingManager, i.e. add access control to `Syndicate.stake`.