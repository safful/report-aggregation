## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Use immutable keyword](https://github.com/code-423n4/2021-06-tracer-findings/issues/29) 

# Handle

gpersoon


# Vulnerability details

## Impact
Some of the contracts set variables in the initialize function that are never changed. See for examples in the "proof of concept" section.
Here the solidity keyword "immutable" could be added to the variables as an extra security measure.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L17
    address public tracer;
    IInsurance public insurance;
    IOracle public oracle;    
    
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Liquidation.sol#L24
    uint256 public override maxSlippage;
    IPricing public pricing;
    ITracerPerpetualSwaps public tracer;
    address public insuranceContract;
    address public fastGasOracle;
        
//https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L20
      address public collateralAsset; // Address of collateral asset
     address public token; // token representation of a users holding in the pool
     ITracerPerpetualSwaps public tracer; // Tracer associated with Insurance Pool
 
## Tools Used

## Recommended Mitigation Steps
Add immutable where possible

