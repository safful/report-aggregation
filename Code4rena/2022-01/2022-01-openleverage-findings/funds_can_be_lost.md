## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Funds can be lost](https://github.com/code-423n4/2022-01-openleverage-findings/issues/220) 

# Handle

csanuragjain


# Vulnerability details

## Impact
User funds can be lost if Admin sets startTimes[i] to 0

## Proof of Concept

1. Navigate to contract https://github.com/code-423n4/2022-01-openleverage/blob/main/openleverage-contracts/contracts/farming/FarmingPools.sol

2. Check the initDistributions function

```
function initDistributions(address[] memory stakeTokens, uint64[] memory startTimes, uint64[] memory durations) external onlyAdmin {
        for (uint256 i = 0; i < stakeTokens.length; i++) {
            require(distributions[stakeTokens[i]].starttime == 0, 'Init once');
            distributions[stakeTokens[i]] = Distribution(durations[i], startTimes[i], 0, 0, 0, 0, 0);
        }
    }
```

3. Assume Admin calls this for token X with startTimes[i] as 0. This creates a new distribution with start time as 0 for token X

4. User Y stakes amount 500 for this token X

5. Admin calls initDistributions again with token X and startTimes[i] as 1000. This overwrites and reinitializes distributions[X] which means totalStaked becomes 0 and contract has lost all track of user funds now

## Recommended Mitigation Steps
Add a check to see startTimes[i]!=0 in initDistributions function

