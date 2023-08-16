## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack Of Return Value Check On the Dex Handler Malt Price Calculation](https://github.com/code-423n4/2021-11-malt-findings/issues/75) 

# Handle

defsec


# Vulnerability details

## Impact

During the code review, It has been seen that malt price return value has not been checked on the function.  If oracle is returned price as a 0, fullReturn will be zero on the earlyExitReturn function.

## Proof of Concept

1. Navigate to "https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/AuctionEscapeHatch.sol#L124"
2. The return value maltMarketPrice() function has not been checked.

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider to add return value check. The maltPrice should be more than zero for the calculation.

"""
require(dexHandler.maltMarketPrice()>0, "Price should be more than zero");
"""

