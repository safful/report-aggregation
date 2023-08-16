## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [todos left in the code](https://github.com/code-423n4/2021-06-tracer-findings/issues/12) 

# Handle

gpersoon


# Vulnerability details

## Impact
There are several todos left in the code.

## Proof of Concept
.\Pricing.sol:                     // todo by using public variables lots of these can be removed
.\Trader.sol:                      // todo this could be succeptible to re-entrancy as
.\lib\LibLiquidation.sol:    // todo with the below * -1, note ints can overflow as 2^-127 is valid but 2^127 is not.
.\lib\LibPrices.sol:             // todo double check safety of this.

## Tools Used

## Recommended Mitigation Steps
Check, fix and remove the todos before it is deployed in production



