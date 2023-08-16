## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [State variables that could be set immutable](https://github.com/code-423n4/2021-12-maple-findings/issues/7) 

# Handle

robee


# Vulnerability details

In the following files there are state variables that could be set immutable to save gas. 
The list of format <solidity file>, <state variable name that could be immutable>: 
There are some variables that I was not sure if are assigned actually twice in real use. I added them anyway.

        DebtLockerStorage.sol, _repossessed
        DebtLockerStorage.sol, _loan
        DebtLockerStorage.sol, _liquidator
        DebtLockerStorage.sol, _pool
        DebtLockerStorage.sol, _allowedSlippage
        DebtLockerStorage.sol, _amountRecovered
        DebtLockerStorage.sol, _fundsToCapture
        DebtLockerStorage.sol, _minRatio
        DebtLockerStorage.sol, _principalRemainingAtLastClaim
        Liquidator.sol, collateralAsset
        Liquidator.sol, destination
        Liquidator.sol, fundsAsset
        Liquidator.sol, owner
        MapleLoanFactory.sol, isLoan
        MapleLoanInternals.sol, _borrower
        MapleLoanInternals.sol, _lender
        MapleLoanInternals.sol, _pendingBorrower
        MapleLoanInternals.sol, _pendingLender
        MapleLoanInternals.sol, _collateralAsset
        MapleLoanInternals.sol, _fundsAsset
        MapleLoanInternals.sol, _collateralRequired
        MapleLoanInternals.sol, _principalRequested
        MapleProxyFactory.sol, mapleGlobals
        MapleProxyFactory.sol, defaultVersion
        MapleProxyFactory.sol, upgradeEnabledForPath
        MapleProxyFactory.t.sol, governor
        MapleProxyFactory.t.sol, notGovernor
        MapleProxyFactory.t.sol, globals
        MapleProxyFactory.t.sol, factory
        MapleProxyFactory.t.sol, implementation1
        MapleProxyFactory.t.sol, implementation2
        MapleProxyFactory.t.sol, initializerV1
        MapleProxyFactory.t.sol, initializerV2
        MapleProxyFactory.t.sol, user




