## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-15

# [User lose his remaining rewards in GiantMevAndFeesPool when new deposits happen because _onDepositETH() set claimed[][] to max without transferring user remaining rewards](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/240) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L195-L204
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantPoolBase.sol#L33-L48


# Vulnerability details

## Impact
When `depositETH()` is called in giant pool it calls `_onDepositETH()` which calls `_setClaimedToMax()` to make sure new ETH stakers are not entitled to ETH earned by but this can cause users to lose their remaining rewards when they deposits. code should first transfer user remaining rewards when deposit happens.

## Proof of Concept
This is `depositETH()` code in `GiantPoolBase`:
```
    /// @notice Add ETH to the ETH LP pool at a rate of 1:1. LPs can always pull out at same rate.
    function depositETH(uint256 _amount) public payable {
        require(msg.value >= MIN_STAKING_AMOUNT, "Minimum not supplied");
        require(msg.value == _amount, "Value equal to amount");

        // The ETH capital has not yet been deployed to a liquid staking network
        idleETH += msg.value;

        // Mint giant LP at ratio of 1:1
        lpTokenETH.mint(msg.sender, msg.value);

        // If anything extra needs to be done
        _onDepositETH();

        emit ETHDeposited(msg.sender, msg.value);
    }
```
As you can see it increase user `lpTokenETH` balance and then calls `_onDepositETH()`. This is `_onDepositETH()` and `_setClaimedToMax()` code in `GiantMevAndFeesPool` contract:
```
    /// @dev On depositing on ETH set claimed to max claim so the new depositor cannot claim ETH that they have not accrued
    function _onDepositETH() internal override {
        _setClaimedToMax(msg.sender);
    }

    /// @dev Internal re-usable method for setting claimed to max for msg.sender
    function _setClaimedToMax(address _user) internal {
        // New ETH stakers are not entitled to ETH earned by
        claimed[_user][address(lpTokenETH)] = (accumulatedETHPerLPShare * lpTokenETH.balanceOf(_user)) / PRECISION;
    }
```
As you can see the code set `claimed[msg.sender][address(lpTokenETH]` to maximum value so the user wouldn't be entitled to previous rewards but if user had some remaining rewards in contract he would lose those rewards can't withdraw them. these are the steps:
1- `user1` deposit `10` ETH to giant pool and `accumulatedETHPerLPShare` value is `2` and `claimed[user1][lpTokenETH]` would be `10 * 2 = 20`.
2- some time passes and `accumulatedETHPerLPShare` set to `4` and `user1` has `10 * 4 - 20 = 20` unclaimed ETH rewards (the formula in the code: `balance * rewardPerShare - claimed`).
3- `user` deposit `5` ETH to giant pool and `accumulatedETHPerLPShare` is `4` so the code would call `_onDepositETH()` which calls `_setClaimedToMax` which sets `claimed[user1][lpTokenETH]` to `15 * 4 = 60`.
4- `user1` new remaining ETH reward would be `15 * 4 - 60 = 0`. and `user1` won't receive his rewards because when he deposits contract don't transfer remaining rewards and set claim to max so user loses his funds.

## Tools Used
VIM

## Recommended Mitigation Steps
when deposit happens contract should first send remaining rewards, then increase the user's balance and then set the user claim to max.