## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas savings of 100 per user by caching _timelockBalances[user] in _sweepTimelockBalances()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/32) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Mapping state variable value _timelockBalances[user] is read on consecutive lines 655 and 656 resulting in 2 SLOADS (2100 + 100 gas). 

Impact: Caching this in a local variable would save ~= 100 gas savings per user iteration (by converting the use of the second 100-gas costing SLOAD to 1 MSTORE and 1 MLOAD both of which only cost 3 gas). If there are 1000 users in a call to sweepTimelockBalances(), this could be significant savings of 100,000 gas.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L655-L656


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache _timelockBalances[user] in a local variable before using on lines 655 and 656.

