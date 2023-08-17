## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-19

# [withdrawETH() in GiantPoolBase don't call _distributeETHRewardsToUserForToken() or _onWithdraw() which would make users to lose their remaining rewards ](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/260) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L50-L64
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L180-L193


# Vulnerability details

## Impact
Function `_distributeETHRewardsToUserForToken()` is used to distribute remaining reward of user and it's called in `_onWithdraw()` of `GiantMevAndFeesPool`. but function `withdrawETH()` in `GiantPoolBase` don't call either of them and burn user giant LP token balance so if user withdraw his funds and has some remaining ETH rewards he would lose those rewards because his balance set to zero.

## Proof of Concept
This is `withdrawETH()` code in `GiantPoolBase`:
```
    /// @notice Allow a user to chose to burn their LP tokens for ETH only if the requested amount is idle and available from the contract
    /// @param _amount of LP tokens user is burning in exchange for same amount of ETH
    function withdrawETH(uint256 _amount) external nonReentrant {
        require(_amount >= MIN_STAKING_AMOUNT, "Invalid amount");
        require(lpTokenETH.balanceOf(msg.sender) >= _amount, "Invalid balance");
        require(idleETH >= _amount, "Come back later or withdraw less ETH");

        idleETH -= _amount;

        lpTokenETH.burn(msg.sender, _amount);
        (bool success,) = msg.sender.call{value: _amount}("");
        require(success, "Failed to transfer ETH");

        emit LPBurnedForETH(msg.sender, _amount);
    }
```
As you can see it burn user `lpTokenETH` balance and don't call either `_distributeETHRewardsToUserForToken()` or `_onWithdraw()`. and in function `claimRewards()` uses `lpTokenETH.balanceOf(msg.sender)` to calculate user rewards so if user balance get to `0` user won't get the remaining rewards.
These are steps that this bug happens:
1. `user1` deposit `10` ETH into the giant pool and `claimed[user1][lpTokenETH]` is `20` and `accumulatedETHPerLPShare` is `2`.
2. some time passes and `accumulatedETHPerLPShare` set to `3`.
3. `user1` unclaimed rewards are `10 * 3 - 20 = 10` ETH.
4. `user1` withdraw his `10` ETH by calling `withdrawETH(10)` and contract set `lpTokenETH` balance of `user1`  to `0` and transfer `10` ETH to user.
5. now if `user1` calls `claimRewards()` he would get `0` reward as his `lpTokenETH` balance is `0`.

so users lose their unclaimed rewards by withdrawing their funds.

## Tools Used
VIM

## Recommended Mitigation Steps
user's unclaimed funds should be calculated and transferred before any actions that change user's balance.