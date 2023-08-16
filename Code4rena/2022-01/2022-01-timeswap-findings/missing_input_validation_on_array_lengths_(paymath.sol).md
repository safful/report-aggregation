## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Missing input validation on array lengths (PayMath.sol)](https://github.com/code-423n4/2022-01-timeswap-findings/issues/137) 

# Handle

ye0lde


# Vulnerability details

## Impact

The function below fails to perform input validation on arrays to verify the lengths match. 
A mismatch could lead to an exception or undefined behavior.

## Proof of Concept

`ids`, `assetsIn` (copied into from `maxAssetsIn` on line 18)
https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/PayMath.sol#L15-L28

While `givenMaxAssetsIn` is an internal function if you trace the code back the parameters are passed in by an external function (`pay` or `payEthAsset` or `payEthCollateral`) with no array length validation.

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

Add input validation to check that the length of all arrays match (`ids`, `maxAssetsIn`).

