## Tags

- bug
- G (Gas Optimization)
- mStableYieldSource
- sponsor confirmed

# [[MStableYieldSource.sol] Public functions that should be declared as external to save gas](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/10) 

# Handle

maplesyrup


# Vulnerability details

## Impact

This is a gas optimization, does not affect the contract negatively, only optimizes it.

## Proof of Concept

According to Slither Analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external), functions that are never called within the contract should be declared as external to save gas for the contract.

In this case, there were only 2 functions in the contract that were found that should be declared as external for further gas optimization.

-----------

Code Snippet:

function approveMax() public {...} <---- should be declared external

(contracts/yield-source/MStableYieldSource.sol, lines #61-65)

function depositToken() public view override returns (address underlyingMasset) {...} <---- should be declared external

(contracts/yield-source/MStableYieldSource.sol, lines #69-71) 

------------

Console output:

INFO:Detectors:
approveMax() should be declared external:
	- MStableYieldSource.approveMax() (contracts/yield-source/MStableYieldSource.sol#61-65)
depositToken() should be declared external:
	- MStableYieldSource.depositToken() (contracts/yield-source/MStableYieldSource.sol#69-71)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external

## Tools Used

PoolTogether Contracts
Solidity (v 0.7.4)
Hardhat (v 2.5.0)
Yarn (v 1.22.10)
Slither Analyzer (v 0.8.0)

## Recommended Mitigation Steps

1. Clone repository for PoolTogether Smart Contracts
2. Create a python virtual environment with a stable python version
3. Install Slither Analyzer on the python VEM
4. Run Slither against all contracts via artifacts

