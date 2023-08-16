## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [withdraw timelock can be circumvented](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/91) 

# Handle

cmichel


# Vulnerability details

One can withdraw the entire `PrizePool` deposit by circumventing the timelock.
Assume the user has no credits for ease of computation:
- user calls `withdrawWithTimelockFrom(user, amount=userBalance)` with their entire balance. This "mints" an equivalent `amount` of `timelock` and resets `_unlockTimestamps[user] = timestamp = blockTime + lockDuration`.
- user calls `withdrawWithTimelockFrom(user, amount=0)` again but this time withdrawing `0` amount. This will return a `lockDuration` of `0` and thus `unlockTimestamp = blockTime`. The inner `_mintTimelock` now resets `_unlockTimestamps[user] = unlockTimestamp`
- As `if (timestamp <= _currentTime()) ` is true, the full users amount is now transferred out to the user in the `_sweepTimelockBalances` call.

## Impact

Users don't need to wait for their deposit to contribute their fair share to the prize pool.
They can join before the awards and leave right after without a penalty which leads to significant issues for the protocol.
It's the superior strategy but it leads to no investments in the strategy to earn the actual interest.

## Recommended Mitigation Steps

The unlock timestamp should be increased by duration each time, instead of being reset to the duration.

