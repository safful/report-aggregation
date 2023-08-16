## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Logic error in fee subtraction](https://github.com/code-423n4/2021-06-tracer-findings/issues/127) 

# Handle

0xsanson


# Vulnerability details

## Impact
In LibBalances.applyTrade() we need to collect a fee from the trade.
The current code however subtracts a fee from the short position and adds it to the long. The correct implementation is to subtract a fee to both (see TracerPerpetualSwaps.sol#L272).
This issue causes withdrawals problems, since Tracer thinks it can withdraw the collect fees, leaving the users with an incorrect amount of quote tokens.

## Proof of Concept
https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibBalances.sol#L187

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Change +fee to -fee in the highlighted line.

