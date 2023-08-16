## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [setHalfDecayPoint check allowed values](https://github.com/code-423n4/2021-10-union-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setHalfDecayPoint allows setting an arbitrary value of halfDecayPoint.
However if halfDecayPoint == 0 then inflationPerBlock will have a division by 0.

Probably it is also useful to have an upper limit for halfDecayPoint.

## Proof of Concept
https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/token/Comptroller.sol#L67-L69

https://github.com/code-423n4/2021-10-union/blob/4176c366986e6d1a6b3f6ec0079ba547b040ac0f/contracts/token/Comptroller.sol#L275-L278


## Tools Used

## Recommended Mitigation Steps
In the function setHalfDecayPoint:
Verify that the new value of halfDecayPoint is within an allowable range ( certainly != 0)

