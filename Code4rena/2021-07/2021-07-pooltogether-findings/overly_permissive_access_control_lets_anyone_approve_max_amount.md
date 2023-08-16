## Tags

- bug
- 1 (Low Risk)
- mStableYieldSource
- sponsor confirmed

# [Overly permissive access control lets anyone approve max amount](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Overly permissive access control to lets anyone approve max amount. This may be ok but is inconsistent with SwappableYieldSource.sol where the similar function is onlyOwner.


## Proof of Concept

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L61-L65

https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L133-L135


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check requirements/spec and ensure this is ok or else add Ownable inheritance to enforce onlyOwner for this function.

