## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Timelock reuse function argument as argument for the event emit](https://github.com/code-423n4/2021-11-malt-findings/issues/214) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Reuse the function argument in the event emit instead of the storage variable. This saves a SLOAD.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L66
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Timelock.sol#L82


## Tools Used

## Recommended Mitigation Steps
- L76 write: emit NewDelay(_delay);
- L92: write: emit NewGracePeriod(_gracePeriod);

