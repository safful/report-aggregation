## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Comment for formula calcEscrowLiquidationAmount different than code](https://github.com/code-423n4/2021-06-tracer-findings/issues/16) 

# Handle

gpersoon


# Vulnerability details

## Impact
The comment for the formula in calcEscrowLiquidationAmount is:
 currentMargin - (minMargin - currentMargin) * portion

however it is coded as:
 {currentMargin - (minMargin - currentMargin)} * portion

According to Ray/Lions mane the comment is wrong

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibLiquidation.sol#L32
//       Calculated as currentMargin - (minMargin - currentMargin) * portion of whole position being liquidated
function calcEscrowLiquidationAmount(
..
        int256 amountToEscrow = currentMargin - (minMargin.toInt256() - currentMargin);
        int256 amountToEscrowProportional = PRBMathSD59x18.mul(amountToEscrow, PRBMathSD59x18.div(amount, totalBase));


## Tools Used

## Recommended Mitigation Steps
Fix the comment

