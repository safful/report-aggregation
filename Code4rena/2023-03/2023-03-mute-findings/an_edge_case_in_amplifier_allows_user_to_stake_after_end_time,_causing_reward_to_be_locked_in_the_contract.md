## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [An edge case in amplifier allows user to stake after end time, causing reward to be locked in the contract](https://github.com/code-423n4/2023-03-mute-findings/issues/23) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L208-L212


# Vulnerability details

## Proof of Concept
Observe that if nobody has staked after the period has ended, it's still possible for a single user to stake even though the period has ended.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L208-L212
```
        if (firstStakeTime == 0) {
            firstStakeTime = block.timestamp;
        } else {
            require(block.timestamp < endTime, "MuteAmplifier::stake: staking is over");
        }
```

The staker can't get any of the rewards because the update modifier won't drip the rewards (since _mostRecentValueCalcTime = firstStakeTime >= endTime).
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L89-L95
```
        if (_mostRecentValueCalcTime == 0) {
            _mostRecentValueCalcTime = firstStakeTime;
        }

        uint256 totalCurrentStake = totalStake();

        if (totalCurrentStake > 0 && _mostRecentValueCalcTime < endTime) {
            ...
        }
```

At the same time, the protocol can't withdraw the rewards with rescueToken either since there is a staker, and no reward has been claimed yet (so the following check fails).
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L187
```
        else if (tokenToRescue == muteToken) {
            if (totalStakers > 0) {
                require(amount <= IERC20(muteToken).balanceOf(address(this)).sub(totalRewards.sub(totalClaimedRewards)),
                    "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
                );
            }
        }
```

## Impact
Suppose the staking period ends and nobody has staked. The admin would like to withdraw the rewards. A malicious user can front-run the rescueTokens call with a call to stake to lock all the rewards inside the contract indefinitely.

## Tools Used
Manual Review

## Recommended Mitigation Steps
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/amplifier/MuteAmplifier.sol#L208-L212
The require shouldn't be inside the else block.