## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization on the Public Function](https://github.com/code-423n4/2021-10-mochi-findings/issues/38) 

# Handle

defsec


# Vulnerability details

## Impact

This does not directly impact the smart contract in anyway besides cost. This is a gas optimization to reduce cost of smart contract. Calling each function, we can see that the public function uses 496 gas, while the external function uses only 261.

## Proof of Concept

According to Slither Analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external), there are functions in the contract that are never called. These functions should be declared as external in order to save gas.

Slither Detector:

external-function:

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVault.sol#L75

## Tools Used

Slither

## Recommended Mitigation Steps

1. Clone repository for Mochi Smart Contracts.
2. Create a python virtual environment with a stable python version.
3. Install Slither Analyzer on the python VEM.
4. Run Slither against all contracts.
5. Define public functions as an external for the gas optimization.

