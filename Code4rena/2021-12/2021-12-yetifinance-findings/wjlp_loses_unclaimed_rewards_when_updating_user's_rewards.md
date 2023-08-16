## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [WJLP loses unclaimed rewards when updating user's rewards](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/141) 

# Handle

kenzo


# Vulnerability details

After updating user's rewards in `_userUpdate`, if the user has not claimed them, and `_userUpdate` is called again (eg. on another `wrap`), the user's unclaimed rewards will lose the previous unclaimed due to wrong calculation.

## Impact
Loss of yield for user.

## Proof of Concept
When updating the user's unclaimedJoeReward, the function doesn't save it's previous value. [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L251:#L253)
```
        if (user.amount > 0) {
            user.unclaimedJOEReward = user.amount.mul(accJoePerShare).div(1e12).sub(user.rewardDebt);
        }
        if (_isDeposit) {
            user.amount = user.amount.add(_amount);
        } else {
            user.amount = user.amount.sub(_amount);
        }
        // update for JOE rewards that are already accounted for in user.unclaimedJOEReward
        user.rewardDebt = user.amount.mul(accJoePerShare).div(1e12);
```
So for example, rewards can be lost in the following scenario. We'll mark "acc1" for the value of "accJoePerShare" at step 1.
1. User Zebulun wraps 100 tokens. After  `_userUpdate` is called: unclaimedJOEReward  = 0, rewardDebt = 100*acc1.
2. Zebulun wraps 50 tokens: unclaimedJOEReward = 100*acc2 - 100*acc1, rewardDebt = 150 * acc2.
3. Zebulun wraps 1 token: unclaimedJOEReward = 150*acc3 - 150*acc2, rewardDebt = 151*acc3
So in the last step, Zebulun's rewards only take into account the change in accJoePerShare in steps 2-3, and lost the unclaimed rewards from steps 1-2.

## Recommended Mitigation Steps
Change the unclaimed rewards calculation to:
```
user.unclaimedJOEReward = user.unclaimedJOEReward.add(user.amount.mul(accJoePerShare).div(1e12).sub(user.rewardDebt));
```

