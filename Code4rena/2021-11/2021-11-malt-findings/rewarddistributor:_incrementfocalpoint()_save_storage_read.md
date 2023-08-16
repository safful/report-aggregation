## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [RewardDistributor:_incrementFocalPoint() save storage read](https://github.com/code-423n4/2021-11-malt-findings/issues/142) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
L185:  focalID = focalID + 1; can be removed and L197 can be adapted to:  _resetFocalPoint(++focalID, newEndTime); to save at least one warm storage read (100 gas)

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/RewardSystem/RewardDistributor.sol#L180
## Tools Used

## Recommended Mitigation Steps

