## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [MovingAverage:initialize reuse argument variable instead storage variable in the loop condition](https://github.com/code-423n4/2021-11-malt-findings/issues/209) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Reuse _sampleMemory instead of the storage variable sampleMemory in the condition statement of the loop to save gas.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L60
## Tools Used

## Recommended Mitigation Steps
- rewrite L60 as: for (uint i = 0; i < _sampleMemory ; i++)

