## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [GiantPool should not check ETH amount on withdrawal](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/92) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L53


# Vulnerability details

## Impact
The `GiantPoolBase.withdrawETH` function requires that the amount to withdraw is at least as big as the `MIN_STAKING_AMOUNT` ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L53](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L53)).  

This check does not serve any purpose and can actually cause the user problems when withdrawing his ETH.  

## Proof of Concept
1. Bob deposits ETH into the GiantPool with the `GiantPoolBase.depositETH` function.  
   The amount is equal to `MIN_STAKING_AMOUNT + 0.99 * MIN_STAKING_AMOUNT`.
2. Bob witdraws `MIN_STAKING_AMOUNT` ETH from the GiantPool.  
3. Bob has `0.99 * MIN_STAKING_AMOUNT` ETH left in the GiantPool. This is a problem since he cannot withdraw this amount of ETH since it is smaller than `MIN_STAKING_AMOUNT`.  
   In order to withdraw his funds, Bob needs to first add funds to the GiantPool such that the deposited amount is big enough for withdrawal.  However this causes extra transaction fees to be paid (loss of funds) and causes a bad user experience.  

## Tools Used
VSCode

## Recommended Mitigation Steps
The `require(_amount >= MIN_STAKING_AMOUNT, "Invalid amount");` statement should just be removed. It does not serve any purpose anyway.  