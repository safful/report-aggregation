## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- SushiYieldSource

# [Caching sushiAddr and sushiBar in local variables to save 200 gas in supplyTokenTo()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/47) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Caching sushiAddr and sushiBar in local variables right at the beginning of supplyTokenTo() (similar to what's done in redeemToken) can save 100 gas from repeat SLOADs for each of them for a total savings of 200.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/SushiYieldSource.sol#L48-L51

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Caching sushiAddr and sushiBar in local variables at the beginning of supplyTokenTo() and use those instead.

