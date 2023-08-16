## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Comment in partialLiquidationIsValid misleading](https://github.com/code-423n4/2021-06-tracer-findings/issues/18) 

# Handle

gpersoon


# Vulnerability details

## Impact
The comments for partialLiquidationIsValid indicate that the params are in WAD format (except liquidationGasCost)
However the parameter minimumLeftoverGasCostMultiplier originates from Liquidation.sol and has the value 10.
So it is not in WAD format and the comment is misleading.

## Proof of Concept
//https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibLiquidation.sol#L149
 @dev Assumes params are WAD except liquidationGasCost
 function partialLiquidationIsValid(
        Balances.Position memory updatedPosition,
        uint256 lastUpdatedGasPrice,
        uint256 liquidationGasCost,
        uint256 price,
        uint256 minimumLeftoverGasCostMultiplier
    ) 

//https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Liquidation.sol#L27
uint256 public override minimumLeftoverGasCostMultiplier = 10;

## Tools Used

## Recommended Mitigation Steps
Update the comment


