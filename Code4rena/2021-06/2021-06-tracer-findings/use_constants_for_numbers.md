## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use constants for numbers](https://github.com/code-423n4/2021-06-tracer-findings/issues/14) 

# Handle

gpersoon


# Vulnerability details

## Impact
At several places constants are hardcoded as numbers. It's more readable and more maintainable to turn them into explicit constants.
That also lowers to risk to change it on one place and forget is on another place.
See examples in proof of concept

## Proof of Concept
.\Insurance.sol:            if (publicCollateralAmount > 10**18) {
.\Insurance.sol:                amount = poolHoldings - 10**18;
.\Insurance.sol:                publicCollateralAmount = 10**18;
.\Insurance.sol:            if (publicCollateralAmount < 10**18) {
.\Insurance.sol:            } else if (poolHoldings - amount < 10**18) {
.\Insurance.sol:                amount = poolHoldings - 10**18;
.\Insurance.sol:                publicCollateralAmount = 10**18;
.\Insurance.sol:            uint256 multiplyFactor = 36523 * (10**11);
.\Insurance.sol:        return tracer.leveragedNotionalValue() / 100;
.\oracle\GasOracle.sol:    uint8 public override decimals = 18;
.\lib\libprices.sol:       for (uint256 i = 0; i < 8; i++) {
.\lib\libprices.sol:           uint256 currTimeWeight = 8 - i;
.\lib\LibPrices.sol:        return (averageTracerPrice.toInt256() - averageOraclePrice.toInt256()) / 90;
.\lib\LibBalances.sol:        uint256 adjustedLiquidationGasCost = liquidationGasCost * 6;
.\lib\LibPerpetuals.sol:        percentFull = percentFull * 100; // To bring it up to the same percentage units as everything else
.\Liquidation.sol:    uint256 public override minimumLeftoverGasCostMultiplier = 10;

## Tools Used

## Recommended Mitigation Steps
Replace numeric values with constants


