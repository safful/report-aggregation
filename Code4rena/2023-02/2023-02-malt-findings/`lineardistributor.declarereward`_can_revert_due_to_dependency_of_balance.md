## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [`LinearDistributor.declareReward` can revert due to dependency of balance](https://github.com/code-423n4/2023-02-malt-findings/issues/35) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L147-L151
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L185-L186
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L123-L136


# Vulnerability details

## Impact
`LinearDistributor.declareReward` will revert and it can cause permanent DOS.

## Proof of Concept

In `LinearDistributor.declareReward`, if the balance is greater than the bufferRequirement, the rest will be forfeited.


```
    if (balance > bufferRequirement) {
      // We have more than the buffer required. Forfeit the rest
      uint256 net = balance - bufferRequirement;
      _forfeit(net);
    }
```

And in `_forfeit`, it requires forfeited (= balance - bufferRequirement) <= declaredBalance.

```
  function _forfeit(uint256 forfeited) internal {
    require(forfeited <= declaredBalance, "Cannot forfeit more than declared");
```

So when an attacker sends some collateral tokens to `LinearDistributor`, the balance will be increased and it can cause revert in `_forfeit` and `declareReward`.

Since `declareReward` sends vested amount before `_forfeit` and the vested amount will be increased by time, so this DOS will be temporary. 

```
    uint256 distributed = (linearBondedValue * netVest) / vestingBondedValue;
    uint256 balance = collateralToken.balanceOf(address(this));

    if (distributed > balance) {
      distributed = balance;
    } 

    if (distributed > 0) {
      // Send vested amount to liquidity mine
      collateralToken.safeTransfer(address(rewardMine), distributed);
      rewardMine.releaseReward(distributed);
    }

    balance = collateralToken.balanceOf(address(this));
```
But if the attacker increases the balance enough to cover all reward amount in vesting, `declareReward` will always revert and it can cause permanent DOS.

`decrementRewards` updates `declaredBalance`, but it only decreases `declaredBalance`, so it can't mitigate the DOS.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Track collateral token balance and add sweep logic for unused collateral tokens in `LinearDistributor`.