## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [AbstractRewardMine.getRewardOwnershipFraction shouldn't be used internally](https://github.com/code-423n4/2021-11-malt-findings/issues/114) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on function calls.

## Proof of Concept

```earned``` function calls public ```getRewardOwnershipFraction``` function:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/AbstractRewardMine.sol#L144

## Recommended Mitigation Steps

Now:
```
function totalDeclaredReward() virtual public view returns (uint256) {
	return rewardToken.balanceOf(address(this));
}
function getRewardOwnershipFraction(address account) public view returns(uint256 numerator, uint256 denominator) {
	numerator = balanceOfRewards(account);
	denominator = totalDeclaredReward();
}
...
function earned(address account) public view returns (uint256 earnedReward) {
	(uint256 rewardNumerator, uint256 rewardDenominator) = getRewardOwnershipFraction(account);

```

To be:
```
function earned(address account) public view returns (uint256 earnedReward) {
	uint256 rewardNumerator = balanceOfRewards(account);
	uint256 rewardDenominator = rewardToken.balanceOf(address(this));
```

