## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`WJLP.getPendingRewards()` should be aview function](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/233) 

# Handle

Ruhum


# Vulnerability details

## Impact
View functions consume less gas. `WJLP.getPendingRewards()` is technically also a view function but not specified as one. Because the `IMasterChefJoeV2` interface used by the contract is wrong. It says `poolInfo()` is not a view function, which it is.

## Proof of Concept
getPendingRewards: https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L190

Faulty interface function: https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L29

actually poolinfo is just an array so its getter is a view function: https://github.com/traderjoe-xyz/joe-core/blob/main/contracts/MasterChefJoeV2.sol#L85

## Tools Used
none

## Recommended Mitigation Step
Declare both functions as `view` to save gas

