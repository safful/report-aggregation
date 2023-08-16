## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use calldata is a little more gas efficient ](https://github.com/code-423n4/2021-07-sherlock-findings/issues/60) 

# Handle

jonah1005


# Vulnerability details

## Impact
Using `calldata` for function parameter is a slightly more gas efficient 

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L192-L225
## Tools Used
None
## Recommended Mitigation Steps
ref: https://mudit.blog/solidity-gas-optimization-tips/


