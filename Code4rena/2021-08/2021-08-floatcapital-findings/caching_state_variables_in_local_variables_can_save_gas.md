## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed
- fixed-in-upstream-repo

# [Caching state variables in local variables can save gas](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/76) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There are multiple places where state variables are reused within functions by loading them multiple times. These operations result in expensive SLOAD instructions where the first SLOAD costs 2100 gas and successive SLOADs of the same variable cost 100 gas (since the Berlin hardfork).

Using local memory variables to cache them will remove the unnecessary SLOADs costing 100 gas resulting in MLOADs that only cost 3 gas units.

## Proof of Concept

Caching latestMarket can save upto 13 SLOADs i.e. 1300 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L261-L298

Caching staked can save upto 1 SLOAD i.e. 100 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L267
https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L276

Caching latestMarket can save upto 3 SLOADs i.e. 3 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L352-L365

Caching marketUpdateIndex[marketIndex] appropriately can save many SLOADs: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L817-L819

https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L677-L757

https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L860-L864

https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/LongShort.sol#L911-L922

Caching longShort address can save 300 300 gas by avoiding 3 SLOADs: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/TokenFactory.sol#L68-L70

Caching amountReservedInCaseOfInsufficientAaveLiquidity can save upto 2 SLOADs i.e. 200 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/YieldManagerAave.sol#L114-L121

Caching paymentToken can save upto 1 SLOAD i.e. 100 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/YieldManagerAave.sol#L132-L142

Caching aaveIncentiveController can save upto 1 SLOAD i.e. 100 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/YieldManagerAave.sol#L162-L167

Caching totalReservedForTreasury can save upto 1 SLOAD i.e. 100 gas: https://github.com/code-423n4/2021-08-floatcapital/blob/bd419abf68e775103df6e40d8f0e8d40156c2f81/contracts/contracts/YieldManagerAave.sol#L162-L167


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Cache storage-based state variables in local memory-based variables appropriately to convert SLOADs to MLOADs and reduce gas consumption from 100 units to 3 units.

