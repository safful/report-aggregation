## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [AuraLocker kick reward only takes last locked amount into consideration, instead of whole balance](https://github.com/code-423n4/2022-05-aura-findings/issues/156) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/contracts/AuraLocker.sol#L404


# Vulnerability details

The issue occurs in AuraLocker, when expired locks are processed via kicking, and if all the user locks have expired.
In this scenario, to calculate the kick reward, `_processExpiredLocks` multiplies the last locked amount by the number of epochs between the last lock's unlock time and the current epoch.
A comment in this section mentions `"wont have the exact reward rate that you would get if looped through"`. However, there's no reason not to multiply *user's whole locked balance* by the number of epochs since the *last lock's* unlock time, *instead of only the last locked amount*.
While this will still not be as accurate as looping through, this will give a more accurate kick reward result, which is still bounded by the full amount that would have been calculated if we had looped through.

## Impact
The reward calculation is inaccurate and lacking for no reason.
Kickers receive less rewards than they should.
Giving them a bigger, more accurate reward, will incentivize them better.

## Proof of Concept
[This](https://github.com/code-423n4/2022-05-aura/blob/main/contracts/AuraLocker.sol#L396:#L405) is the section that calculates the kick reward if all locks have expired:
```
            //check for kick reward
            //this wont have the exact reward rate that you would get if looped through
            //but this section is supposed to be for quick and easy low gas processing of all locks
            //we'll assume that if the reward was good enough someone would have processed at an earlier epoch
            if (_checkDelay > 0) {
                uint256 currentEpoch = block.timestamp.sub(_checkDelay).div(rewardsDuration).mul(rewardsDuration);
                uint256 epochsover = currentEpoch.sub(uint256(locks[length - 1].unlockTime)).div(rewardsDuration);
                uint256 rRate = AuraMath.min(kickRewardPerEpoch.mul(epochsover + 1), denominator);
                reward = uint256(locks[length - 1].amount).mul(rRate).div(denominator);
            }
```
This flow is for low gas processing, so the function is not looping through all the locks (unlike the flow where some locks have not expired yet).
In this flow, the function is just calculating the reward for the last lock.

Instead of doing this, it can multiply the *total amount locked by the user* (`locked`, already saved) by the *number of epochs between the last unlock time and current epoch*.
The reward will still be smaller than if we had looped through all the rewards (since then each lock amount would be multiplied by more than just the last lock's number of expired epochs).
But it would be more accurate and give better incentive for kicking.


## Recommended Mitigation Steps
Change the last line in the code above to:
```
                reward = uint256(locked).mul(rRate).div(denominator);
```
This will keep the low gas consumption of this flow, while giving a more accurate result.

