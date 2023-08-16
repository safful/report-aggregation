## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused state variables](https://github.com/code-423n4/2022-01-insure-findings/issues/3) 

# Handle

robee


# Vulnerability details

Unused state variables are gas consuming at deployment (since they are located in storage) and are 
a bad code practice. Removing those variables will decrease deployment gas cost and improve code quality. 
This is a full list of all the unused storage variables we found in your code base. 
The format is <solidity file>, <unused storage variable name>: 

        IndexTemplate.sol, pendingEnd
        BondingPremium.sol, ADJUSTER


