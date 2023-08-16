## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching badger and badgerSett can save 400 gas in supplyTokenTo()](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/38) 

# Handle

0xRajeev


# Vulnerability details

## Impact

State variables badger and badgerSett addresses are read two and four times respectively in supplyTokenTo(). Caching them in local variables at the beginning of the function and using those local variables can save 400 gas from avoiding 3 repeated SLOADs for badgerSett and 1 repeated SLOAD for badger.

Impact: Gas savings of 400

## Proof of Concept

Two badger reads: https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/BadgerYieldSource.sol#L44-L45

Four badgerSett reads: 

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/BadgerYieldSource.sol#L45

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/BadgerYieldSource.sol#L47

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/BadgerYieldSource.sol#L48

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/BadgerYieldSource.sol#L49

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache badger and badgerSett state variables in local variables at the beginning of the function and use those local variables instead.

