## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [NatSpec typo in `_doSherX` @return](https://github.com/code-423n4/2021-07-sherlock-findings/issues/125) 

# Handle

0xsanson


# Vulnerability details

## Impact
In the function `_doSherX` in Payout.sol, the natSpec comment @return states that `sherUsd` is the 'Total amount of USD of the underlying tokens that are being transferred'. I think that's a typo, and it's supposed to be the amount *excluded* from being transferred.

## Proof of Concept
Payout.sol L71

## Tools Used
editor

## Recommended Mitigation Steps
Correct the statement.

