## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing check for toPool != fromPool](https://github.com/code-423n4/2021-07-spartan-findings/issues/127) 

# Handle

0xRajeev


# Vulnerability details

## Impact

zapLiquidity() used to trade LP tokens of one pool to another is missing a check for toPool != fromPool which may happen accidentally. The check will prevent unnecessary transfers and avoid any fees/slippage or accounting errors.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L58-L71


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add toPool != fromPool as part of input validation.

