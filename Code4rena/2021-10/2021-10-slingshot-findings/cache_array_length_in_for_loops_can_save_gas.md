## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache array length in for loops can save gas](https://github.com/code-423n4/2021-10-slingshot-findings/issues/63) 

# Handle

WatchPug


# Vulnerability details

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Instances include:

- `ModuleRegistry.sol#registerSwapModuleBatch()`

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/ModuleRegistry.sol#L46-L46

- `ModuleRegistry.sol#unregisterSwapModuleBatch()`

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/ModuleRegistry.sol#L61-L61

- `Executioner.sol#executeTrades()`

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Executioner.sol#L34-L34

- `Slingshot.sol#executeTrades()`

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Slingshot.sol#L75-L76

