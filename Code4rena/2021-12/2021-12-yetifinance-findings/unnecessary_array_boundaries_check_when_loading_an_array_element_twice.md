## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary array boundaries check when loading an array element twice](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/11) 

# Handle

robee


# Vulnerability details

There are places in the code (especially in for-each loops) that loads the same array element more
than once. In such cases, only one array boundaries check should take place, and the rest are unnecessary.
Therefore, this array element should be cached in a local variable and then be loaded 
again using this local variable, skipping the redundent second array boundaries check: 

        BorrowerOperations.sol, variable name: _colls times: 2 at: _requireNoDuplicateColls
        BorrowerOperations.sol, variable name: _indices times: 2 at: _requireRouterAVAXIndicesInOrder
        DefaultPool.sol, variable name: _amounts times: 2 at: sendCollsToActivePool
        YetiCustomBase.sol, variable name: _amounts times: 2 at: _subColls
        StabilityPool.sol, variable name: amounts times: 2 at: _sendGainsToDepositor
        TroveManager.sol, variable name: lastCollError_Redistribution times: 2 at: redistributeDebtAndColl
        TroveManager.sol, variable name: lastYUSDDebtError_Redistribution times: 2 at: redistributeDebtAndColl
        TroveManager.sol, variable name: totalStakes times: 2 at: redistributeDebtAndColl
        TroveManager.sol, variable name: allColls times: 2 at: _closeTrove



