## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Remove unnecessary address cast in Vault.sol](https://github.com/code-423n4/2022-01-insure-findings/issues/159) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The Vault.sol contract contains several state variables of type address. There is no need to cast these variable to type address because they are already of type address. Removing the cast function can save gas.

## Proof of Concept

The token address state variable is unnecessarily cast to address type in two places in Vault.sol:
- [Line 350](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L350)
- [Line 467](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L467)

## Recommended Mitigation Steps

Remove the unnecessary address cast from address variables.

