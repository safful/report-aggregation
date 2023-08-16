## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [BASIS_POINTS naming convention](https://github.com/code-423n4/2021-06-gro-findings/issues/23) 

# Handle

gpersoon


# Vulnerability details

## Impact
The variable BASIS_POINTS in Buoy3Pool.sol is written in capitals, which is the naming convention for constants.
However BASIS_POINTS isn't a constant, because it is updated in setBasisPointsLmit
This is confusing when reading the code.

## Proof of Concept
https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pools/oracle/Buoy3Pool.sol#L30
uint256 public BASIS_POINTS = 20;

 function setBasisPointsLmit(uint256 newLimit) external onlyOwner {
        uint256 oldLimit = BASIS_POINTS;
        BASIS_POINTS = newLimit;
        emit LogNewBasisPointLimit(oldLimit, newLimit);
    }

## Tools Used

## Recommended Mitigation Steps
Change BASIS_POINTS  to something like:
basisPoints


