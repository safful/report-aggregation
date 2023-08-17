## Tags

- bug
- duplicate
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [massUpdatePools() is susceptible to DoS with block gas limit](https://github.com/code-423n4/2022-05-aura-findings/issues/197) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/convex-platform/contracts/contracts/ConvexMasterChef.sol#L178-L183


# Vulnerability details

## Impact
massUpdatePools() is a public function and it calls the updatePool() function for the length of poolInfo. Hence, it is an unbounded loop, depending on the length of poolInfo.
If poolInfo.length is big enough, block gas limit may be hit.

## Proof of Concept
https://consensys.github.io/smart-contract-best-practices/attacks/denial-of-service/#dos-with-block-gas-limit

## Tools Used
Manual analysis

## Recommended Mitigation Steps
I suggest to limit the max number of loop iterations to prevent hitting block gas limit.


