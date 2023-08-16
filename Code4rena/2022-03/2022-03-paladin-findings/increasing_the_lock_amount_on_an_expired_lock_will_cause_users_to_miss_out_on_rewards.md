## Tags

- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Increasing the Lock Amount on an Expired Lock Will Cause Users to Miss Out on Rewards](https://github.com/code-423n4/2022-03-paladin-findings/issues/95) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L284-L294

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1137-L1233

# Vulnerability details

## Impact

Paladin protocol allows users to increase the amount or duration of their lock while it is stil active. Increasing the amount of an active lock should only increase the total locked amount and it shouldn't make any changes to the associated bonus ratios as the duration remains unchanged. 

However, if a user increases the lock amount on an expired lock, a new lock will be created with the duration of the previous lock and the provided non-zero amount. Because the `action != LockAction.INCREASE_AMOUNT` check later on in the function does not hold true, `userCurrentBonusRatio` will contain the last updated value from the previous lock. As a result, the user will not receive any rewards for their active lock and they will need to increase the duration of the lock to fix lock's bonus ratio.

## Recommended Mitigation Steps

Consider preventing users from increasing the amount on an expired lock. This should help to mitigate this issue.
