## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- reviewed

# [function lockFunds in TopUpActionLibrary can cause serious fund lose. fee and Capped bypass. It's not calling stakerVault.increaseActionLockedBalance when transfers stakes.](https://github.com/code-423n4/2022-04-backd-findings/issues/60) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L57-L65


# Vulnerability details

## Impact
In function TopUpActionLibrary.lockFunds when transfers stakes from payer it doesn't call stakerVault.increaseActionLockedBalance for that payer so stakerVault.actionLockedBalances[payer] is not get updated for payer and stakerVault.stakedAndActionLockedBalanceOf(payer) is going to show wrong value and any calculation based on this function is gonna be wrong which will cause fund lose and theft and some restriction bypasses.

## Proof of Concept
When user wants to create a TopUpAction. so he seposit his funds to Pool and get LP token. then stake the LP token in StakerVault and use that stakes to create a TopUp position with function TopUpAction.register. This function transfer user stakes (locks user staks) and create his position.
for transferring and locking user stakes it uses TopUpActionLibrary.lockFunds. function lockFunds transfers user stakes but don't call stakerVault.increaseActionLockedBalance for the payer which cause that stakerVault.actionLockedBalances[payer] to get different values(not equal to position.depositTokenBalance).
function StakerVault.stakedAndActionLockedBalanceOf(account) uses stakerVault.actionLockedBalances[account] so it will return wrong value and any where in code that uses stakedAndActionLockedBalanceOf() is going to cause problems.
three part of the codes uses stakerVault.stakedAndActionLockedBalanceOf():
1- LiqudityPool.depositFor() for checking user total deposits to be less than depositCap.
2- LiqudityPool._updateUserFeesOnDeposit() for updating user fee on new deposits.
3- userCheckpoint() for calculating user rewards.
attacker can use #1 and #2 to bypass high fee payment and max depositCap and #3 will cause users to lose 
rewards.

The detail steps:
1- user deposit fund to Pool and get LP token.
2- user stakes LP token in StakerVault.
3- user approve TopUpAction address to transfer his staks in StakerVault.
3- user use all his stakes to create a position with TopUpAction.register() function.
3.1- register() will call lockFunds to transfer and lock user stakes.
3.2- lockFunds() will transfer user stakes with stakerVault.transferFrom() but don't call stakerVault.increaseActionLockedBalance() so StakerVault.actionLockedBalances[user] will be zero.
3.3- StakerVault.balance[useer] will be zero too because his stakes get transfers in 3.2
4- StakerVault.stakedAndActionLockedBalanceOf(user) will return zero (user has some locked stakes in TopUpAction but because of the bug calculation get out of sync)

In this moment user will lose all the rewards that are minted in LpGauge. becasue userCheckpoint() use stakerVault.stakedAndActionLockedBalanceOf(user) for calculating rewards which is zero  and new rewards will be zero too.

Attacker can use this process to bypass "max deposit Cap" and deposit any amount of assets he wants. because LiqudityPool.depositFor(address,uint256,uint256) uses stakedAndActionLockedBalanceOf to check user deposits which is zero so Attacker can deposit & stake & register to make his balance zero and repeat this and in the end reset his TopUp positions to get back his large stakes which are multiple time bigger than "max deposit Cap"

Attacker can also use this process to bypass fee penalties for early withdraw. because LiqudityPool._updateUserFeesOnDeposit() to get user current balance use stakedAndActionLockedBalanceOf() which is zero. so the value of shareExisting variable become zero and newFeeRatio will be calculated based on feeOnDeposit which can be minFee if asset is already in wallet for some time.


## Tools Used
VIM

## Recommended Mitigation Steps
add this line to TopUpActionLibrary.lockFunds() after stakerVault.transferFrom():

stakerVault.increaseActionLockedBalance(payer, amountLeft);

