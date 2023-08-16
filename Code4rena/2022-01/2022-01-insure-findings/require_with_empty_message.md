## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Require with empty message](https://github.com/code-423n4/2022-01-insure-findings/issues/15) 

# Handle

robee


# Vulnerability details

The following requires are with empty messages. 
This is very important to add a message for any require. Such that the user has enough 
information to know the reason of failure: 

        Solidity file: CDSTemplate.sol, In line 253 with Empty Require message.
        Solidity file: Factory.sol, In line 100 with Empty Require message.
        Solidity file: IndexTemplate.sol, In line 477 with Empty Require message.
        Solidity file: PoolTemplate.sol, In line 929 with Empty Require message.
        Solidity file: Vault.sol, In line 66 with Empty Require message.
        Solidity file: Vault.sol, In line 67 with Empty Require message.
        Solidity file: Vault.sol, In line 68 with Empty Require message.


