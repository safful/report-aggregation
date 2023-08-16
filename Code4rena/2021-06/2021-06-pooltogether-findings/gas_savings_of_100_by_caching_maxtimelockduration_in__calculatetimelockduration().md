## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings of 100 by caching maxTimelockDuration in _calculateTimelockDuration()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/33) 

# Handle

0xRajeev


# Vulnerability details

## Impact

State variable maxTimelockDuration is read twice on consecutive lines 723 and 724 of function _calculateTimelockDuration(). Caching it in a local variable will save 100 gas.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L723-L724


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache maxTimelockDuration in a local variable in the beginning of the function.

