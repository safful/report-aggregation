## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache array length in for loops can save gas](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/128) 

# Handle

WatchPug


# Vulnerability details

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Instances include:

https://github.com/sushiswap/trident/blob/master/contracts/TridentRouter.sol#L214-L234

- minWithdrawals.length
- withdrawnLiquidity.length

https://github.com/sushiswap/trident/blob/master/contracts/TridentRouter.sol#L274

- tokenInput.length

https://github.com/sushiswap/trident/blob/master/contracts/pool/IndexPool.sol#L114-L136

- tokens.length

