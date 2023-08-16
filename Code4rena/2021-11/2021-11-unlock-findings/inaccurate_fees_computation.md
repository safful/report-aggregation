## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Inaccurate fees computation](https://github.com/code-423n4/2021-11-unlock-findings/issues/165) 

# Handle

cmichel


# Vulnerability details

The `MixinTransfer.shareKey` function wants to compute a fee such that `time + fee * time == timeRemaining (timePlusFee)`:

```solidity
uint fee = getTransferFee(keyOwner, _timeShared);
uint timePlusFee = _timeShared + fee;
```

However, if the time remaining is less than the computed fee time, **the computation changes and a different formula is applied**.
The fee is now simply taken on the remaining time.

```solidity
if(timePlusFee < timeRemaining) {
  // now we can safely set the time
  time = _timeShared;
  // deduct time from parent key, including transfer fee
  _timeMachine(_tokenId, timePlusFee, false);
} else {
  // we have to recalculate the fee here
  fee = getTransferFee(keyOwner, timeRemaining);
  // @audit want it such that time + fee * time == timeRemaining, but fee is taken on timeRemaining instead of time
  time = timeRemaining - fee;
}
```

It should compute the `time` without fee as `time = timeRemaining / (1.0 + fee_as_decimal)` instead, i.e., `time = BASIS_POINTS_DEN * timeRemaining / (transferFeeBasisPoints + BASIS_POINTS_DEN)`.

#### POC
To demonstrate the difference with a 10% fee and a `_timeShared = 10,000s` which should be credited to the `to` account.

The correct time plus fee which is reduced from `from` (as in the `timePlusFee < timeRemaining` branch) would be `10,000 + 10% * 10,000 = 11,000`.

However, if `from` has not enough time remaining and `timePlusFee >= timeRemaining`, the entire time remaining is reduced from `from` but the credited `time` is computed wrongly as:
(Let's assume `timeRemaining == timePlusFee`): `time = 11,000 - 10% * 11,000 = 11,000 - 1,100 = 9900`.

They would receive 100 seconds less than what they are owed.

## Impact
When transferring more time than the `from` account has, the credited time is scaled down wrongly and the receiver receives less time (a larger fee is applied).

## Recommended Mitigation Steps
It should change the first `if` branch condition to `timePlusFee <= timeRemaining` (less than or equal).
In the `else` branch, it should compute the time without fee as `time = BASIS_POINTS_DEN * timeRemaining / (transferFeeBasisPoints + BASIS_POINTS_DEN)`.

