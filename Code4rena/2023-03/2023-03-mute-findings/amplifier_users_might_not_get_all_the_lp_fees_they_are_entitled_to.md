## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-06

# [Amplifier users might not get all the LP fees they are entitled to](https://github.com/code-423n4/2023-03-mute-findings/issues/21) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L111


# Vulnerability details

## Proof of Concept
Observe that there is only one place that the amplifier is calling claimFees, and it's inside an if statement of the update modifier, requiring `_mostRecentValueCalcTime < endTime`.

https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L111
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

Consider the following situation. An user X has staked a large amount of LP tokens, and a user Y has staked a normal amount.

Y withdraws as soon as the staking period ends (block.timestamp > endTime), triggering the update modifier, which sets `_mostRecentValueCalcTime = block.timestamp` > endTime. Observe that after this point, the amplifier will never call claimFees again since `_mostRecentValueCalcTime < endTime` will forever be false. 

Meanwhile, X forgot about it, and doesn't withdraw until say 2 weeks after endTime. When X calls withdraw, X won't get the LP fees for those 2 weeks. In fact, nobody will - they are trapped inside the mute switch pair forever since the amplifier won't call claim.


## Impact
Some LP fees can be trapped inside the mute switch pair when it should really be going to the amplifier users.


## Tools Used
Manual Review

## Recommended Mitigation Steps
I believe it's best to move the [LP fee calculation](https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L111-L117) out of the if statement.