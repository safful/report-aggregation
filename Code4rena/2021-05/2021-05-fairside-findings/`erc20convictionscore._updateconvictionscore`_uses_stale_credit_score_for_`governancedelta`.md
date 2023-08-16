## Tags

- bug
- question
- 3 (High Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [`ERC20ConvictionScore._updateConvictionScore` uses stale credit score for `governanceDelta`](https://github.com/code-423n4/2021-05-fairside-findings/issues/41) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
In `ERC20ConvictionScore._updateConvictionScore`, when the user does not fulfill the governance criteria anymore, the `governanceDelta` is the old conviction score of the previous block.

```solidity
isGovernance[user] = false;
governanceDelta = getPriorConvictionScore(
    user,
    block.number - 1
);
```

The user could increase their conviction / governance score first in the same block and then lose their status in a second transaction, and the total governance conviction score would only be reduced by the previous score.

Example:
Block n - 10000: User is a governor and has a credit score of 1000 which was also contributed to the `TOTAL_GOVERNANCE_SCORE`
Block n:
- User updates their own conviction score using public `updateConvictionScore` function which increases the credit score by 5000 based on the accumulated time. The total governance credit score increased by 5000, making the user contribute 6000 credit score to governance in total.
- User transfers their whole balance away, the balance drops below `governanceMinimumBalance` and user is not a governor anymore. The `governanceDelta` update of the transfer should be 6000 (user's whole credit score) but it's only `1000` because it takes the snapshot of block n - 1.

## Impact
The `TOTAL_GOVERNANCE_SCORE` score can be inflated this way and break the voting mechanism in the worst case as no proposals can reach the quorum (percentage of `totalVotes`) anymore.

## Recommended Mitigation Steps
Use the current conviction store which should be `governanceDelta = checkpoints[user][userCheckpointsLength - 1].convictionScore`


