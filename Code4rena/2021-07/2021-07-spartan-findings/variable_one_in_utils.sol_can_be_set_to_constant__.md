## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Variable one in Utils.sol can be set to constant  ](https://github.com/code-423n4/2021-07-spartan-findings/issues/146) 

# Handle

maplesyrup


# Vulnerability details

## Impact
Gas optimizations
Does not affect the contract in any harmful way. Suggestions allow for smart contract gas optimizations.

## Proof of Concept
According to Slither analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-constant), the variable in contract Utils.sol called "one" or Utils.one can be set to a constant as it is considered a variable that does not change throughout the contract. 

Slither Detectors:

constable-states:

Utils.one (contracts/Utils.sol, lines#11) should be constant

------------

Code in contract:

uint public one = 10**18; <---- can be constant as it does not change

--------------

Console output (via Slither in JSON format):

  "constable-states": [
    "Utils.one (contracts/Utils.sol#11) should be constant\n"
  ],

## Tools Used

Spartan Contracts
Solidity (v 0.8.3)
Slither Analyzer (v 0.8.0)

## Recommended Mitigation Steps

1. Clone repository for Spartan Smart Contracts
2. Create a python virtual environment with a stable python version
3. Install Slither Analyzer on the python VEM
4. Run Slither against all contracts

