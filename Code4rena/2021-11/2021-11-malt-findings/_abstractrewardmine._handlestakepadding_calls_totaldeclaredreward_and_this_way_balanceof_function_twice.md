## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ AbstractRewardMine._handleStakePadding calls totalDeclaredReward and this way balanceOf function twice](https://github.com/code-423n4/2021-11-malt-findings/issues/128) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on access and function calls.

## Proof of Concept

totalDeclaredReward is called by _handleStakePadding twice:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AbstractRewardMine.sol#L180

While totalDeclaredReward does expensive balanceOf call:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AbstractRewardMine.sol#L97

## Recommended Mitigation Steps

It is viable to at least remove its double usage:

Now:
```
uint256 totalRewardedWithStakePadding = totalDeclaredReward().add(totalStakePadding());
...
uint256 newStakePadding = bondedTotal == 0 ?
	totalDeclaredReward() == 0 ? amount.mul(INITIAL_STAKE_SHARE_MULTIPLE) : 0 :
	totalRewardedWithStakePadding.mul(amount).div(bondedTotal);
```

To be:
```
uint256 declaredRewardTotal =  rewardToken.balanceOf(address(this));
uint256 totalRewardedWithStakePadding = declaredRewardTotal.add(totalStakePadding());
...
uint256 newStakePadding = bondedTotal == 0 ?
	declaredRewardTotal == 0 ? amount.mul(INITIAL_STAKE_SHARE_MULTIPLE) : 0 : totalRewardedWithStakePadding.mul(amount).div(bondedTotal);
```

