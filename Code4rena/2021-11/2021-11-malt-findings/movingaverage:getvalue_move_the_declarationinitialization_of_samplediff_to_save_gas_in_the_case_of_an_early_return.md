## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [MovingAverage:getValue move the declaration/initialization of sampleDiff to save gas in the case of an early return](https://github.com/code-423n4/2021-11-malt-findings/issues/210) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
move the sampleDiff (L86) below the if statement L88 to save the declaration/initialization of sampleDiff in the case the if block gets executed and the function returns early

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L86
## Tools Used

## Recommended Mitigation Steps
- move the declaration/initialization of of sampleDiff below the if statement

