## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Function purgeDeployer() should be declared external in BondVault.sol](https://github.com/code-423n4/2021-07-spartan-findings/issues/145) 

# Handle

maplesyrup


# Vulnerability details

## Impact

Gas Optimization
This does not directly impact the smart contract in anyway besides cost. This is a gas optimization to reduce cost of smart contract.

## Proof of Concept
According to Slither Analyzer documentation (https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external), there are functions in the contract that are never called. These functions should be declared as external in order to save gas. 

Slither Detector:

external-function:

purgeDeployer() should be declared external:

BondVault.purgeDeployer() (contracts/BondVault.sol, lines#50-52)

-----------------------

Console output (via Slither in JSON format):

"external-function": [
    "purgeDeployer() should be declared external:\n\t- BondVault.purgeDeployer() (contracts/BondVault.sol#50-52)\n",
    "hasMinority(uint256) should be declared external:\n\t- Dao.hasMinority(uint256) (contracts/Dao.sol#601-610)\n",
    "ROUTER() should be declared external:\n\t- Dao.ROUTER() (contracts/Dao.sol#615-621)\n",
    "UTILS() should be declared external:\n\t- Dao.UTILS() (contracts/Dao.sol#624-630)\n",
    "BONDVAULT() should be declared external:\n\t- Dao.BONDVAULT() (contracts/Dao.sol#633-639)\n",
    "DAOVAULT() should be declared external:\n\t- Dao.DAOVAULT() (contracts/Dao.sol#642-648)\n",
    "POOLFACTORY() should be declared external:\n\t- Dao.POOLFACTORY() (contracts/Dao.sol#651-657)\n",
    "SYNTHFACTORY() should be declared external:\n\t- Dao.SYNTHFACTORY() (contracts/Dao.sol#660-666)\n",
    "RESERVE() should be declared external:\n\t- Dao.RESERVE() (contracts/Dao.sol#669-675)\n",
    "SYNTHVAULT() should be declared external:\n\t- Dao.SYNTHVAULT() (contracts/Dao.sol#678-684)\n",
    "greet() should be declared external:\n\t- Greeter.greet() (contracts/Greeter.sol#15-17)\n",
    "setGreeting(string) should be declared external:\n\t- Greeter.setGreeting(string) (contracts/Greeter.sol#19-22)\n"
  ]

## Tools Used

Spartan Contracts
Solidity (v 0.8.3)
Slither Analyzer (v 0.8.0)

## Recommended Mitigation Steps

1. Clone repository for Spartan Smart Contracts
2. Create a python virtual environment with a stable python version
3. Install Slither Analyzer on the python VEM
4. Run Slither against all contracts

