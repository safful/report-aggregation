## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [MovingAverage:getValueWithLookback move sampleDiff  to save gas](https://github.com/code-423n4/2021-11-malt-findings/issues/212) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Move the initialization of sampleDiff below the if block to save gas in the case of return of the if block.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L128

## Tools Used

