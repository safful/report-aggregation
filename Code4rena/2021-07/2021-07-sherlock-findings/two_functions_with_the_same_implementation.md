## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Two functions with the same implementation](https://github.com/code-423n4/2021-07-sherlock-findings/issues/17) 

# Handle

gpersoon


# Vulnerability details

## Impact
The functions getTotalUsdPool and viewAccrueUSDPool have the same implementation.
It saves some gas on the deployment to integrate these functions.
Also the maintenance will be a bit easier.

## Proof of Concept
//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/SherX.sol#L42
function getTotalUsdPool() external view override returns (uint256) {
    SherXStorage.Base storage sx = SherXStorage.sx();
    return sx.totalUsdPool.add(block.number.sub(sx.totalUsdLastSettled).mul(sx.totalUsdPerBlock));
  }

//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherX.sol#L18
  function viewAccrueUSDPool() public view returns (uint256 totalUsdPool) {
    SherXStorage.Base storage sx = SherXStorage.sx();
    totalUsdPool = sx.totalUsdPool.add(block.number.sub(sx.totalUsdLastSettled).mul(sx.totalUsdPerBlock));
  }

## Tools Used

## Recommended Mitigation Steps
Integrate the functions getTotalUsdPool and viewAccrueUSDPool
(e.g. keep one and remove the other and update the references)

