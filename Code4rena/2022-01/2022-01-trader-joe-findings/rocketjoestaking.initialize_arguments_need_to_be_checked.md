## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [RocketJoeStaking.initialize arguments need to be checked](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/266) 

# Handle

hyh


# Vulnerability details

## Impact

Being instantiated with wrong configuration the contract will be inoperable.

If a misconfiguration is noticed too late the various types of malfunctions become possible.

## Proof of Concept

RocketJoeStaking.initialize doesn't check input parameters, which are immutable due to initializer pattern:

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeStaking.sol#L72-75


## Recommended Mitigation Steps

Consider checking joe, rJoe addresses and lastRewardTimestamp to be non-zero and also checking rJoePerSec to be within pre specified bounds

