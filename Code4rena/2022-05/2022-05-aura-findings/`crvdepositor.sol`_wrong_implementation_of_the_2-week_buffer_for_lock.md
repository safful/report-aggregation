## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`CrvDepositor.sol` Wrong implementation of the 2-week buffer for lock](https://github.com/code-423n4/2022-05-aura-findings/issues/343) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/CrvDepositor.sol#L127-L134


# Vulnerability details

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/CrvDepositor.sol#L127-L134

```solidity
uint256 unlockAt = block.timestamp + MAXTIME;
uint256 unlockInWeeks = (unlockAt/WEEK)*WEEK;

//increase time too if over 2 week buffer
if(unlockInWeeks.sub(unlockTime) > 2){
    IStaker(staker).increaseTime(unlockAt);
    unlockTime = unlockInWeeks;
}
```

In `_lockCurve()`, `unlockInWeeks - unlockTime` is being used as a number in weeks, while it actually is a number in seconds.

Thus, comparing it with `2` actually means a 2 seconds buffer instead of a 2 weeks buffer.

The intention is to wait for 2 weeks before extending the lock time again, but the current implementation allows the extension of the lock once a new week begins.

### Recommendation

Consider changing the name of `unlockTime` to `unlockTimeInWeeks`, and:

1. Change L94-102 to:

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/CrvDepositor.sol#L94-L102

```solidity
uint256 unlockAt = block.timestamp + MAXTIME;
uint256 unlockInWeeks = unlockAt / WEEK;

//release old lock if exists
IStaker(staker).release();
//create new lock
uint256 crvBalanceStaker = IERC20(crvBpt).balanceOf(staker);
IStaker(staker).createLock(crvBalanceStaker, unlockAt);
unlockTimeInWeeks = unlockInWeeks;
```

2. Change L127-L134 to:

```solidity
uint256 unlockAt = block.timestamp + MAXTIME;
uint256 unlockInWeeks = unlockAt / WEEK;

//increase time too if over 2 week buffer
if(unlockInWeeks.sub(unlockTime) > 2){
    IStaker(staker).increaseTime(unlockAt);
    unlockTimeInWeeks = unlockInWeeks;
}
```

