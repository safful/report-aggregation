## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-06

# [Withdrawing wrong LPToken from GiantPool leads to loss of funds](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/98) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L95


# Vulnerability details

## Impact
The `GiantPoolBase.withdrawLPTokens` function ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69)) allows to withdraw LP tokens from a GiantPool by burning an equal amount of GiantLP.  

This allows a user to handle the LP tokens directly without the need for a GiantPool as intermediary.  

It is not checked however whether the LP tokens to be withdrawn were transferred to the GiantPool in exchange for staking ETH.  

I.e. whether the LP token are of any value.  

There are two issues associated with this behavior.  

1. A malicious user can create and mint his own LP Token and send it to the GiantPool. Users that want to withdraw LP tokens from the GiantPool can then be tricked into withdrawing worthless attacker LP tokens, thereby burning their GiantLP tokens that are mapped 1:1 to 
ETH. (-> loss of funds)  

2. This can also mess up internal accounting logic. For every LP token that is owned by a GiantPool there should be a corresponding GiantLP token. Using the described behavior this ratio can be broken such that there are LP token owned by the GiantPool for which there is no GiantLP token. This means some LP token cannot be transferred from the GiantPool and there will always be some amount of LP token "stuck" in the GiantPool.  

## Proof of Concept
1. The attacker deploys his own LPToken contract and sends a huge amount of LP tokens to the GiantPool to pass the check in `GiantPoolBase._assertUserHasEnoughGiantLPToClaimVaultLP` ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L95](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L95)). 
2. The attacker tricks Bob into withdrawing the malicious LP tokens from the GiantPool ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L69)). 
3. Bob's GiantLP tokens are burnt and he receives worthless LP tokens.

The same issue exists for the `GiantSavETHVaultPool.withdrawDETH` function.  
But in this case, the victim must also provide a wrong savETHVault address which makes this issue less likely to be exploited.  

## Tools Used
VSCode

## Recommended Mitigation Steps
The GiantPool should store information about which LP tokens it receives for staking ETH. 
When calling the `GiantPoolBase.withdrawLPTokens` function it can then be checked if the LP tokens to be withdrawn were indeed transferred to the GiantPool in exchange for staking ETH.