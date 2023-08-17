## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-01

# [Award is still distributed when there aren't any stakers, allowing users to get reward without staking](https://github.com/code-423n4/2023-03-mute-findings/issues/41) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L95-L118


# Vulnerability details

## Proof of Concept
Consider the update modifier for the amplifier.
```
modifier update() {
        if (_mostRecentValueCalcTime == 0) {
            _mostRecentValueCalcTime = firstStakeTime;
        }

        uint256 totalCurrentStake = totalStake();

        if (totalCurrentStake > 0 && _mostRecentValueCalcTime < endTime) {
            uint256 value = 0;
            uint256 sinceLastCalc = block.timestamp.sub(_mostRecentValueCalcTime);
            uint256 perSecondReward = totalRewards.div(endTime.sub(firstStakeTime));

            if (block.timestamp < endTime) {
                value = sinceLastCalc.mul(perSecondReward);
            } else {
                uint256 sinceEndTime = block.timestamp.sub(endTime);
                value = (sinceLastCalc.sub(sinceEndTime)).mul(perSecondReward);
            }

            _totalWeight = _totalWeight.add(value.mul(10**18).div(totalCurrentStake));

            _mostRecentValueCalcTime = block.timestamp;

            (uint fee0, uint fee1) = IMuteSwitchPairDynamic(lpToken).claimFees();

            _totalWeightFee0 = _totalWeightFee0.add(fee0.mul(10**18).div(totalCurrentStake));
            _totalWeightFee1 = _totalWeightFee1.add(fee1.mul(10**18).div(totalCurrentStake));

            totalFees0 = totalFees0.add(fee0);
            totalFees1 = totalFees1.add(fee1);
        }

        _;
    }
```

Suppose there's been a period with totalCurrentStake = 0, and a user stakes and immediately withdraws in the same transaction.
When the user stakes, update doesn't do anything (including updating _mostRecentValueCalcTime) since totalCurrentStake = 0, and _userWeighted[account] gets set to _totalWeight (which hasn't been updated) in the stake function.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L349
```
function _stake(uint256 lpTokenIn, address account) private {
        ...
        _userWeighted[account] = _totalWeight;
        ...
    }
```
When the user withdraws, totalCurrentStake is no longer zero. Since _mostRecentValueCalcTime wasn't updated when the user staked (since totalCurrentStake was 0), the reward from the period with no stakers gets added to _totalWeight.

`uint256 sinceLastCalc = block.timestamp.sub(_mostRecentValueCalcTime);`
`value = sinceLastCalc.mul(perSecondReward);`
`_totalWeight = _totalWeight.add(value.mul(10**18).div(totalCurrentStake));`
(These are lines in the update modifier)

As a result, this user who staked and immediately withdrew, got all the reward from the period with no stakers. See the reward calculation:
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L371
```
function _applyReward(address account) private returns (uint256 lpTokenOut, uint256 reward, uint256 remainder, uint256 fee0, uint256 fee1) {
        ...
        reward = lpTokenOut.mul(_totalWeight.sub(_userWeighted[account])).div(calculateMultiplier(account, true));
        ...
    }
```

Observe that the exploit condition is met as soon as the staking period starts (as long as nobody stakes immediately). The code attempts to prevent this situation setting _mostRecentValueCalcTime to firstStakeTime the first time that update is invoked (which must be a stake call).
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L89-L91
```
if (_mostRecentValueCalcTime == 0) {
            _mostRecentValueCalcTime = firstStakeTime;
        }
```

        
However, this doesn't do anything since the user can first set the firstStakeTime by staking as soon as the staking period starts, and then make totalCurrentStake 0 by immediately withdrawing. In fact, observe that this takes away from other staker's rewards since perSecondReward is now lower.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L98
`uint256 perSecondReward = totalRewards.div(endTime.sub(firstStakeTime));`

Please add the following test to the "advance to start time" context in amplifier.ts (add it [here](https://github.com/code-423n4/2023-03-mute/blob/main/test/amplifier.ts#L157)), and run with `npm run test-amplifier`.
```
it("get reward without staking", async function () {
        await this.lpToken.transfer(this.staker1.address, staker1Initial.toFixed(), {from: this.owner.address});
        await this.lpToken.connect(this.staker1).approve(
          this.amplifier.address,
          staker1Initial.toFixed()
        );
        
        console.log("dmute balance before: " + await this.dMuteToken.balanceOf(this.staker1.address))
        await this.amplifier.connect(this.staker1).stake(1);
        await this.amplifier.connect(this.staker1).withdraw();
        await time.increaseTo(staking_end-10);
        await this.amplifier.connect(this.staker1).stake(1);
        await this.amplifier.connect(this.staker1).withdraw();
        console.log("dmute balance after: " + await this.dMuteToken.balanceOf(this.staker1.address))
      });

      it("get reward with staking", async function () {
        await this.lpToken.transfer(this.staker1.address, staker1Initial.toFixed(), {from: this.owner.address});
        await this.lpToken.connect(this.staker1).approve(
          this.amplifier.address,
          staker1Initial.toFixed()
        );
        console.log("dmute balance before: " + await this.dMuteToken.balanceOf(this.staker1.address))
        await this.amplifier.connect(this.staker1).stake(1);
        //await this.amplifier.connect(this.staker1).withdraw();
        await time.increaseTo(staking_end-10);
        //await this.amplifier.connect(this.staker1).stake(1);
        await this.amplifier.connect(this.staker1).withdraw();
        console.log("dmute balance after: " + await this.dMuteToken.balanceOf(this.staker1.address))
      });
```
Result on my side:
```
advance to start time
        ✓ reverts without tokens approved for staking
dmute balance before: 0
dmute balance after: 576922930658129099273
        ✓ get reward without staking
dmute balance before: 0
dmute balance after: 576922912276064160247
        ✓ get reward with staking
```

This POC is a bit on the extreme side to get the point across. In the first test, the user stakes and then immediately unstakes, while in the second test, the user stakes for the entire period. In the end, the user gets roughly the same amount of reward.


## Impact
After periods with no stakers, users can get reward without staking. This is also possible at the beginning of the staking period, and doing so then will reduce the reward for other users in the process. 

## Tools Used
Manual review, Hardhat

## Recommended Mitigation Steps
Possible solution 1: set a minimum duration that a user must stake for (prevent them from staking and immediately withdrawing)
Possible solution 2: always update _mostRecentValueCalcTime (regardless totalCurrentStake). i.e. move the following line out of the if statement.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L109

Keep in mind that with solution 2, no one gets the rewards in periods with no stakers - this means that the rescueTokens function needs to be updated to get retrieve these rewards.
