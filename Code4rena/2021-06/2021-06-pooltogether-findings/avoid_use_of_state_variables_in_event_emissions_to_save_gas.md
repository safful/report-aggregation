## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid use of state variables in event emissions to save gas](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/26) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Where possible, use equivalent function parameters or local variables in event emits instead of state variables to prevent expensive SLOADs. Post-Berlin, SLOADs on state variables accessed first-time in a transaction increased from 800 gas to 2100, which is a 2.5x increase.

## Proof of Concept

The Initialized event in PrizePool uses state variables maxExitFeeMantissa and maxTimelockDuration instead of using the equivalent function parameters _maxExitFeeMantissa and _maxTimelockDuration which were just used to set these state variables. Using them instead will save 2 extra SLOADs, leading to gas savings of 200.

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L239-L243

The StakePrizePoolInitialized event uses state variable stakeToken instead of the function parameter _stakeToken used to set it. Using that instead will save 100 gas.

  https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/StakePrizePool.sol#L36-L38 

The IdleYieldSourceInitialized similarly uses idleToken instead of _idleToken.  

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/IdleYieldSource.sol#L62-L66

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use equivalent function parameters or local variables in event emits instead of state variables.

