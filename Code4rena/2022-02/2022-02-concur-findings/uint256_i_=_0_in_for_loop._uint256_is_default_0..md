## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [uint256 i = 0 in for loop. uint256 is default 0.](https://github.com/code-423n4/2022-02-concur-findings/issues/58) 

# Handle

kenta


# Vulnerability details

## Impact
The default value of uint256 is 0 and we do not initialize it and save a little bit of gas. 

## Proof of Concept
https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L121

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L219

## Tools Used

## Recommended Mitigation Steps
for (unit256 i; i < length; i++) {}

