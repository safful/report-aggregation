## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Two Steps Verification before Transferring Ownership](https://github.com/code-423n4/2021-12-sublime-findings/issues/95) 

# Handle

robee


# Vulnerability details


The following contracts have a function that allows them an admin to change it to a different address. If the admin accidentally uses an invalid address for which they do not have the private key, then the system gets locked.
It is important to have two steps admin change where the first is announcing a pending new admin and the new address should then claim its ownership. 
A similar issue was reported in a previous contest and was assigned a severity of medium: [code-423n4/2021-06-realitycards-findings#105](https://github.com/code-423n4/2021-06-realitycards-findings/issues/105) 

        ILendingPoolAddressesProvider.sol
        IUniswapV3Factory.sol
        Controller.sol
        Strategy.sol
        yVault.sol

