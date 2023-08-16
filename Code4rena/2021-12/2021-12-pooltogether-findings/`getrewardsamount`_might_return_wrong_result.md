## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`getRewardsAmount` might return wrong result](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/80) 

# Handle

certora


# Vulnerability details

getRewardsAmount gets epochs ids as uint256[]. However, it should be uint8[].

In _calculateRewardAmount, the epoch start time and end time are calculated:
```
uint256 _epochStartTimestamp = _promotion.startTimestamp + (_epochDuration * _epochId);
uint256 _epochEndTimestamp = _epochStartTimestamp + _epochDuration;
```

and then are casted to uint64 for the rest of the function.
if it's greater than 2**64, it will be truncated.
## Impact
`getRewardsAmount` might return wrong result

## Tools Used
manual review
## Recommended Mitigation Steps
get _epochIds as uint8[] instead uint256[]

