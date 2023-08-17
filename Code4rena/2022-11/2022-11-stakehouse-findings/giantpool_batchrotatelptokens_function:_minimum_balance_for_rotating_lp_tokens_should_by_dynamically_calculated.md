## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-13

# [GiantPool batchRotateLPTokens function: Minimum balance for rotating LP Tokens should by dynamically calculated](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/149) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L127
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L116
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L22


# Vulnerability details

## Impact
The `GiantSavETHVaultPool` and `GiantMevAndFeesPool` both have a `batchRotateLPTokens` function that allows to move staked ETH to another key.  

Both functions require that the GiantLP balance of the sender is `>=0.5 ether`.  

[https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L127](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L127)  

[https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L116](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L116)  

The reason for this is that there is a `common interest` needed in order to rotate LP Tokens.  

The way this is implemented right now does not serve this purpose and even makes the functions unable to be called in some cases.  

The `MIN_STAKING_AMOUNT` for the GiantPools is `0.001 ether` ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L22](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L22)).  

So a user should expect that this amount is sufficient to properly use the contract.  

However even if there are multiple users paying into the GiantPool, they might not reach the 0.5 ETH threshold to call the function.  

So even if they would use some kind of multisig wallet to call the `batchRotateLPTokens` function, it would not be possible.  

Also the threshold does not scale.  

Imagine that User A puts 100 ETH into the GiantPool. Another User B puts 0.5 ETH into the GiantPool.  

Can we speak of "common interest" when User B wants to rotate the LP Tokens?  

## Tools Used
VSCode

## Recommended Mitigation Steps
My suggestion is to use a formula like:  
`require(lpTokenETH.balanceOf(msg.sender) >= (lpTokenETH.totalSupply() / CONSTANT_VALUE))`.  
Where you can choose a CONSTANT_VALUE like 20 or 50.  

This properly scales the required amount and helps mitigate both scenarios.  