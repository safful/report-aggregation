## Tags

- bug
- G (Gas Optimization)
- mStableYieldSource
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/37) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In redeemToken(), given that mAssetBalanceAfter will always be >= mAssetBalanceBefore, using the unchecked directive (solc 0.8.2 has default overflow/underflow checks) on L106 can save bit of gas from the unnecessary (in this case) internal underflow checks on the subtraction.

## Proof of Concept

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L2

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L106

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L100-L106

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change to unchecked {mAssetsActual = mAssetBalanceAfter - mAssetBalanceBefore;}

