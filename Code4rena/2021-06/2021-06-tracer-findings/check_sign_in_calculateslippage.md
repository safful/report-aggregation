## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [check sign in calculateSlippage](https://github.com/code-423n4/2021-06-tracer-findings/issues/17) 

# Handle

gpersoon


# Vulnerability details

## Impact
In function calculateSlippage of LibLiquidation.sol, the value of amountToReturn is calculated by subtracting to numbers.
Later on it is check if this value is negative.
However amountToReturn is an unsigned integer so it can never be negative. If a negative number would be attempted to be assigned, the code will revert,
because solidity 0.8 checks for this.

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibLiquidation.sol#L106
function calculateSlippage(
...
            uint256 amountToReturn = 0;
            uint256 percentSlippage = 0;
            if (avgPrice < receipt.price && receipt.liquidationSide == Perpetuals.Side.Long) {
                amountToReturn = amountExpectedFor - amountSoldFor;
            } else if (avgPrice > receipt.price && receipt.liquidationSide == Perpetuals.Side.Short) {
                amountToReturn = amountSoldFor - amountExpectedFor;
            }
            if (amountToReturn <= 0) {    // can never be smaller than 0, because amountToReturn is uint256
                return 0;
            }

## Tools Used

## Recommended Mitigation Steps
Double check if amountToReturn could be negative. If this is the case change the type of amountToReturn to int256 and add the appropriate type casts


