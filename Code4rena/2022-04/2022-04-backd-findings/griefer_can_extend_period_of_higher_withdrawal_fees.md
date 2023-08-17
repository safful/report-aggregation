## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [Griefer can extend period of higher withdrawal fees](https://github.com/code-423n4/2022-04-backd-findings/issues/56) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L790-L792


# Vulnerability details

## Impact
The `_updateUserFeesOnDeposit()` function in `LiquidityPool.sol` is used to update a user's withdrawal fees after an action such as deposit, transfer in, etc. The withdrawal fee decays toward a minimum withdrawal fee over a period of 1 or 2 weeks (discussed with developer). Since anyone can transfer lp tokens to any user, a griefer can transfer 1 wei of lp tokens to another user to reset their `lastActionTimestamp` used in the withdrawal fee calculation.

The developers nicely weight the updated withdrawal fee by taking the original balance/original fee vs the added balance/added fee. The attacker will only be able to extend the runway of the withdrawal fee cooldown by resetting the `lastActionTimestamp` for future calculations. Example below:

## Proof of Concept
Assumptions:
- MinWithdrawalFee = 0% //For easy math
- MaxWithdrawalFee = 10%
- timeToWait = 2 weeks

### Steps
- User A has `100 wei` of shares
- User A waits 1 week (Current withdrawal fee = 5%)
- User B deposits, receives `1 wei` of shares, current withdrawal fee = 10%
- User B immediately transfers `1 wei` of shares to User A

Based on the formula to calculated User A's new feeRatio:

```
        uint256 newFeeRatio = shareExisting.scaledMul(newCurrentFeeRatio) +
            shareAdded.scaledMul(feeOnDeposit);
```

In reality, User A's withdrawal fee will only increase by a negligible amount since the shares added were very small in proportion to the original shares. We can assume user A's current withdrawal fee is still 5%.

The issue is that the function then reset's User A's `lastActionTimestamp` to the current time. This means that User A will have to wait the maximum 2 weeks for the withdrawal fee to reduce from 5% to 0%. Effectively the cooldown runway is the same length as the original runway length, so the decay down to 0% will take twice as long.

`meta.lastActionTimestamp = uint64(_getTime());`

## Tools Used
Manual Review

## Recommended Mitigation Steps
Instead of resetting `lastActionTimestamp` to the current time, scale it the same way the `feeRatio` is scaled. I understand that this would technically not be the timestamp of the last action, so the variable would probably need to be renamed.

