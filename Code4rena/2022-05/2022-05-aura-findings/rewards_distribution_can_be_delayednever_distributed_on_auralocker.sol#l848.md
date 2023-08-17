## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor acknowledged
- sponsor confirmed

# [Rewards distribution can be delayed/never distributed on AuraLocker.sol#L848](https://github.com/code-423n4/2022-05-aura-findings/issues/1) 

# Lines of code

https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L848


# Vulnerability details



Rewards distribution can be delayed/never distributed on [AuraLocker.sol#L848 ](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L848)


### Issue

Someone malicious can delay the rewards distribution for non `cvxCrv` tokens distributed on AuraLocker.sol.


1: Attacker will send one wei of token that are distributed on the [AuraLocker.sol ](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol) to [AuraStakingProxy](https://github.com/aurafinance/aura-contracts-lite/blob/6d60fca6f821dca1854a538807e7928ee582553a/contracts/AuraStakingProxy.sol).

2: Attacker will call [distributeOther](https://github.com/aurafinance/aura-contracts-lite/blob/6d60fca6f821dca1854a538807e7928ee582553a/contracts/AuraStakingProxy.sol#L203).
The function will call notifyRewardAmount that calls [_notifyReward](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L860)


When calling [_notifyReward](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L860) the rewards left to distribute over the 7 days are redistributed throughout a new period starting immediately.

```
uint256 remaining = uint256(rdata.periodFinish).sub(block.timestamp);
uint256 leftover = remaining.mul(rdata.rewardRate);
rdata.rewardRate = _reward.add(leftover).div(rewardsDuration).to96();
```

_Example:_ If the reward rate is 1 token (10**18) per second and 3.5 days are left (302400 seconds), we get a leftover of 302400 tokens. this is then divided by 604800, the reward rate is now 0.5 and the user of the protocol will have to wait one week for tokens that were supposed to be distributed over 3.5 days. This can be repeated again and again so that some rewards are never distributed. 


### Suggestion
I can see that [queueNewRewards](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L820) has some protective mechanism. A new period is started only if the token that is added on top of the already distributed tokens during the duration is over 120%.

I suggest adding a similar check to [queueNewRewards](https://github.com/aurafinance/aura-contracts-lite/blob/main/contracts/AuraLocker.sol#L820) 

