## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [AbstractRewardMine._handleStakePadding logic cases can be separated and function simplified](https://github.com/code-423n4/2021-11-malt-findings/issues/131) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on operations.

## Proof of Concept

The function contains two non-intersecting logic pathways, which can be separated to lighten calculations.

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AbstractRewardMine.sol#L179


## Recommended Mitigation Steps

Now:
```
function _handleStakePadding(address account, uint256 amount) internal {
	uint256 totalRewardedWithStakePadding = totalDeclaredReward().add(totalStakePadding());

	uint256 INITIAL_STAKE_SHARE_MULTIPLE = 1e6;

	uint256 bondedTotal = totalBonded();

	uint256 newStakePadding = bondedTotal == 0 ?
		totalDeclaredReward() == 0 ? amount.mul(INITIAL_STAKE_SHARE_MULTIPLE) : 0 :
		totalRewardedWithStakePadding.mul(amount).div(bondedTotal);

	_addToStakePadding(account, newStakePadding);
}
```

To be:
```
function _handleStakePadding(address account, uint256 amount) internal {
	uint256 bondedTotal = totalBonded();
	
	uint256 newStakePadding;
	if (bondedTotal == 0) {
		uint256 INITIAL_STAKE_SHARE_MULTIPLE = 1e6;
		newStakePadding = totalDeclaredReward() == 0 ? amount.mul(INITIAL_STAKE_SHARE_MULTIPLE) : 0;
	} else {
		newStakePadding = (totalDeclaredReward().add(totalStakePadding())).mul(amount).div(bondedTotal);
	}

	if (newStakePadding > 0)
		_addToStakePadding(account, newStakePadding);
}
```

