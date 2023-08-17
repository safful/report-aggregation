## Tags

- bug
- 3 (High Risk)
- judge review requested
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-21

# [`bringUnusedETHBackIntoGiantPool` in `GiantMevAndFeesPool` can be used to steal `LPTokens`](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/366) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L126


# Vulnerability details

## Impact
real `LPTokens` can be transferred out of `GiantMevAndFeesPool` through fake `_stakingFundsVaults` provided by an attacker.
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L126

## Proof of Concept
`bringUnusedETHBackIntoGiantPool` takes in `_stakingFundsVaults`, `_oldLPTokens`, `_newLPTokens` and rotate `_amounts` from old to new tokens. The tokens are thoroughly verified by `burnLPForETH` in `ETHPoolLPFactory`. 
However, theres is no checking for the validity of `_stakingFundsVaults`, nor the relationship between `LPTokens` and `_stakingFundsVaults`. Therefore, an attacker can create fake contracts for `_stakingFundsVaults`, with `burnLPTokensForETH`, that takes `LPTokens` as parameters. The `msg.sender` in `burnLPTokensForETH` is `GiantMevAndFeesPool`, thus the attacker can transfer `LPTokens` that belongs to `GiantMevAndFeesPool` to any addresses it controls.

## Tools Used
manual

## Recommended Mitigation Steps
Always passing liquid staking manager address, checking its real and then requesting either the savETH vault or staking funds vault is a good idea to prove the validity of vaults. 
