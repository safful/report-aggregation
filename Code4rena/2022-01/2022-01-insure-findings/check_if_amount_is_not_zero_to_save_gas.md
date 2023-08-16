## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Check if amount is not zero to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/13) 

# Handle

robee


# Vulnerability details

The following functions could skip other steps if the amount is 0. (A similar issue: https://github.com/code-423n4/2021-10-badgerdao-findings/issues/82) 

        InsureDAOERC20.sol, name
        InsureDAOERC20.sol, symbol
        InsureDAOERC20.sol, decimals
        InsureDAOERC20.sol, totalSupply
        InsureDAOERC20.sol, balanceOf
        InsureDAOERC20.sol, transfer
        InsureDAOERC20.sol, allowance
        InsureDAOERC20.sol, approve
        InsureDAOERC20.sol, transferFrom
        InsureDAOERC20.sol, increaseAllowance
        InsureDAOERC20.sol, decreaseAllowance
        InsureDAOERC20.sol, _transfer
        InsureDAOERC20.sol, _mint
        InsureDAOERC20.sol, _burn
        InsureDAOERC20.sol, _approve
        InsureDAOERC20.sol, _beforeTokenTransfer
        InsureDAOERC20.sol, _afterTokenTransfer

