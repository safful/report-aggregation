## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Declare state variables as immutable](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/162) 

# Handle

p4st13r4


# Vulnerability details

## Impact

In `WJLP.sol`, state variables `JLP` and `JOE` are initialized in the constructor and never reassigned again. Thus, they can be declared `immutable` rather than `constant` in order to save gas

## Proof of Concept

[https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L41](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L41)

## Tools Used

Editor

