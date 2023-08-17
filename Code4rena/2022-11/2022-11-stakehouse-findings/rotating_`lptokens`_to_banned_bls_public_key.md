## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-02

# [Rotating `LPTokens` to banned BLS public key](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/64) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/ETHPoolLPFactory.sol#L76


# Vulnerability details

## Impact

It is possible to rotate `LPTokens` to a banned BLS public key. This is not a safe action, because it can result in insolvency of the project (specially if the banned BLS public key was malicious).

## Proof of Concept

When a user deposits ETH for staking by calling `depositETHForStaking`, the manager checks whether the provided BLS public key is banned or not.
`require(liquidStakingNetworkManager.isBLSPublicKeyBanned(_blsPublicKeyOfKnot) == false, "BLS public key is banned or not a part of LSD network");`
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/StakingFundsVault.sol#L113

If it is not banned the `LPToken` related to that BLS public key will be minted to the caller, so the number of `LPToken` related to that BLS public key will be increased. 
https://github.com/code-423n4/2022-11-stakehouse/blob/39a3a84615725b7b2ce296861352117793e4c853/contracts/liquid-staking/ETHPoolLPFactory.sol#L125

If it is banned, it will not be possible to stake to this BLS public key, so the number of `LPToken` will not be increased. But the issue is that it is still possible to increase the `LPToken` of this BLS public key through rotating `LPToken`. 

In other words, a malicious user can call `rotateLPTokens`, so that the `_oldLPToken` will be migrated to `_newLPToken` which is equal to the `LPToken` related to the banned BLS public key.

In summary, the vulnerability is that during rorating `LPTokens`, it is not checked that the `_newLPToken` is related to a banned BLS public key or not.

## Tools Used

## Recommended Mitigation Steps
The following line should be added to function `rotateLPTokens(...)`:
`require(liquidStakingNetworkManager.isBLSPublicKeyBanned(blsPublicKeyOfNewKnot ) == false, "BLS public key is banned or not a part of LSD network");`
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/ETHPoolLPFactory.sol#L76