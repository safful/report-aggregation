## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [`ERC20ConvictionScore` allows transfers to special TOTAL_GOVERNANCE_SCORE address](https://github.com/code-423n4/2021-05-fairside-findings/issues/42) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The credit score of the special `address(type(uint160).max)` is supposed to represent the sum of the credit scores of all users that are governors.
But any user can directly transfer to this address increasing its balance and accumulating a credit score in `_updateConvictionScore(to=address(uint160.max), amount)`.
It'll first write a snapshot of this address' balance which should be very low:

```solidity
// in _updateConvictionScore
_writeCheckpoint(user, userNum, userNew) = _writeCheckpoint(TOTAL_GOVERNANCE_SCORE, userNum, checkpoints[user][userNum - 1].convictionScore + convictionDelta);
```

This address then accumulates a score based on its balance which can be updated using `updateConvictionScore(uint160.max)` and breaks the invariant.

## Impact
Increasing it might be useful for non-governors that don't pass the voting threshold and want to grief the proposal voting system by increasing the `quorumVotes` threshold required for proposals to pass. (by manipulating `FairSideDAO.totalVotes`). `totalVotes` can be arbitrarily inflated and break the voting mechanism as no proposals can reach the quorum (percentage of `totalVotes`) anymore.

## Recommended Mitigation Steps
Disallow transfers from/to this address. Or better, track the total governance credit score in a separate variable, not in an address.


