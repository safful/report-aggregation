## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [use of floating pragma ](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/42) 

# Handle

JMukesh


# Vulnerability details

## Impact
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

https://swcregistry.io/docs/SWC-103

## Proof of Concept
https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L1

## Tools Used
manual review

## Recommended Mitigation Steps
use fixed solidity version

