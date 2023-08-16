## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings of 300 by caching _currentAwardBalance in captureAwardBalance()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/29) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Cache _currentAwardBalance state variable in a local variable for computation to save gas. 4 SLOADs + 1 SSTORE can be reduced to 1 SLOAD and 1 STORE. 

Impact: Saves 300 gas from avoid 3 SLOADs because each SLOAD to already accessed storage slot costs 100.

## Proof of Concept

2 SLOADs: https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L456

1 SSTORE + 1 SLOAD: https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L465

1 SLOAD: https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L470


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache _currentAwardBalance in a local variable in the beginning, use that for computation/return and one updation to state variable at the end.

