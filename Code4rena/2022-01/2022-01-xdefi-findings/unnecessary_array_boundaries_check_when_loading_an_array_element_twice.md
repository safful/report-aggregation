## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary array boundaries check when loading an array element twice](https://github.com/code-423n4/2022-01-xdefi-findings/issues/8) 

# Handle

robee


# Vulnerability details

There are places in the code (especially in for-each loops) that loads the same array element more
than once. In such cases, only one array boundaries check should take place, and the rest are unnecessary.
Therefore, this array element should be cached in a local variable and then be loaded 
again using this local variable, skipping the redundent second array boundaries check: 

        XDEFIDistributionHelper.sol, variable name: tokenIds times: 3 at: getAllLockedPositionsForAccount


