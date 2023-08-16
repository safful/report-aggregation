## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Tokens can be burned with no access control](https://github.com/code-423n4/2022-01-insure-findings/issues/158) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The Vault.sol contract has two address state variables, the `keeper` variable and the `controller` variable, which are both permitted to be the zero address. If both variables are zero simultaneously, any address can burn the available funds (available funds = balance - totalDebt) by sending these tokens to the zero address with the unprotected `utilitize()` function. If a user has no totalDebt, the user can lose their entire underlying token balance because of this.

## Proof of Concept

The problematic `utilize()` function is [found here](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L342-L352). To see how the two preconditions can occur:
1. The keeper state variable is only changed by the `setKeeper()` function [found here](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L502). If this function is not called, the keeper variable will retain the default value of address(0), which bypasses [the only access control for the utilize function](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L344).
2. There is a comment [here on line 69](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L502https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L502) stating the controller state variable can be zero. There is no zero address check for the controller state variable in the Vault constructor.

If both address variables are left at their defaults of address(0), then the safeTransfer() call [on line 348](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L348) would send the tokens to address(0).

## Recommended Mitigation Steps

Add the following line to the very beginning of the `utilize()` function:
`require(address(controller) != address(0))`

This check is already found in many other functions in Vault.sol, including the `_unutilize()` function.

