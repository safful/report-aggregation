## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Save `vault.getToken()` as an immutable variable in `YaxisVaultAdapter.sol` contract can save gas](https://github.com/code-423n4/2021-11-yaxis-findings/issues/52) 

# Handle

WatchPug


# Vulnerability details

Across the functions in `YaxisVaultAdapter.sol`, `vault.getToken()` is called many times, each one will cost a significant amount of gas due to external call.

Given that the result of `vault.getToken()` will never change, create an immutable variable named `token` in the contract and replace `vault.getToken()` with `token` can save gas.

`vault.getLPToken()` is a similar situation, it can also be cached as an immutable variable.

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L45-L45

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L70-L70

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L76-L76

