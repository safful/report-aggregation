## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [`ERC20ConvictionScore`'s `governanceDelta` should be subtracted when user is not a governor anymore](https://github.com/code-423n4/2021-05-fairside-findings/issues/40) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `TOTAL_GOVERNANCE_SCORE` is supposed to track the sum of the credit scores of all governors.

In `ERC20ConvictionScore._updateConvictionScore`, when the user does not fulfill the governance criteria anymore and is therefore removed, the `governanceDelta` should be negative but it's positive.

```solidity
isGovernance[user] = false;
governanceDelta = getPriorConvictionScore(
    user,
    block.number - 1
);
```

It then gets added to the new total:

```solidity
uint224 totalGCSNew =
    add224(
        totalGCSOld,
        governanceDelta,
        "ERC20ConvictionScore::_updateConvictionTotals: conviction score amount overflows"
    );
```

## Impact
The `TOTAL_GOVERNANCE_SCORE` tracks wrong data leading to issues throughout all contracts like wrong `FairSideDAO.totalVotes` data which can then be used for anyone to pass proposals in the worst case.
Or `totalVotes` can be arbitrarily inflated and break the voting mechanism as no proposals can reach the quorum (percentage of `totalVotes`) anymore.

## Recommended Mitigation Steps
Return a negative, signed integer for this case and add it to the new total.


