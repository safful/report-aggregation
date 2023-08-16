## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong reward token calculation in MasterChef contract](https://github.com/code-423n4/2022-02-concur-findings/issues/219) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/MasterChef.sol#L86


# Vulnerability details

## Impact
When adding new token pool for staking in MasterChef contract
```javascript
function add(address _token, uint _allocationPoints, uint16 _depositFee, uint _startBlock)
```
All other, already added, pools should be updated but currently they are not.
Instead, only totalPoints is updated. Therefore, old (and not updated) pools will lose it's share during the next update.
Therefore, user rewards are not computed correctly (will be always smaller).

## Proof of Concept
Scenario 1:
1. Owner adds new pool (first pool) for staking with points = 100 (totalPoints=100)
   and 1 block later Alice stakes 10 tokens in the first pool.
2. 1 week passes
3. Alice withdraws her 10 tokens and claims X amount of reward tokens.
   and 1 block later Bob stakes 10 tokens in the first pool.
4. 1 week passes
5. Owner adds new pool (second pool) for staking with points = 100 (totalPoints=200)
   and 1 block later Bob withdraws his 10 tokens and claims X/2 amount of reward tokens.
   But he should get X amount

Scenario 2:
1. Owner adds new pool (first pool) for staking with points = 100 (totalPoints=100).
2. 1 block later Alice, Bob and Charlie stake 10 tokens there (at the same time).
3. 1 week passes
4. Owner adds new pool (second pool) for staking with points = 400 (totalPoints=500)
5. Right after that, when Alice, Bob or Charlie wants to withdraw tokens and claim rewards they will only be able to claim 20% of what they should be eligible for, because their pool is updated with 20% (100/500) rewards instead of 100% (100/100) rewards for the past week.

## Tools Used
Manual review

## Recommended Mitigation Steps
Update all existing pools before adding new pool. Use the massUdpate() function which is already present ... but unused.

