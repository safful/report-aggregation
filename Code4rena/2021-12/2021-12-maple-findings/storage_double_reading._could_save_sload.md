## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed

# [Storage double reading. Could save SLOAD](https://github.com/code-423n4/2021-12-maple-findings/issues/4) 

# Handle

robee


# Vulnerability details

Reading a storage variable is gas costly (SLOAD). In cases of multiple read of a storage variable in the same scope, caching the first read (i.e saving as a local variable) can save gas and decrease the
 overall gas uses. The following is a list of functions and the storage variables that you read twice: 

        Liquidator.sol Variable _locked is read 2 times in the function:  liquidatePortion
        MapleLoanInternals.sol Variable _drawableFunds is read 2 times in the function:  _closeLoan
        MapleLoanInternals.sol Variable _refinanceCommitment is read 2 times in the function:  _acceptNewTerms
        MapleLoanInternals.sol Variable _fundsAsset is read 3 times in the function:  _fundLoan
        MapleProxyFactory.t.sol Variable governor is read 2 times in the function:  setUp
        MapleProxyFactory.t.sol Variable globals is read 2 times in the function:  setUp
        MapleProxyFactory.t.sol Variable implementation1 is read 3 times in the function:  test_registerImplementation
        MapleProxyFactory.t.sol Variable initializerV1 is read 2 times in the function:  test_registerImplementation
        MapleProxyFactory.t.sol Variable factory is read 3 times in the function:  test_setDefaultVersion
        MapleProxyFactory.t.sol Variable implementation1 is read 2 times in the function:  test_setDefaultVersion
        MapleProxyFactory.t.sol Variable factory is read 4 times in the function:  test_createInstance
        MapleProxyFactory.t.sol Variable implementation1 is read 3 times in the function:  test_createInstance
        MapleProxyFactory.t.sol Variable factory is read 5 times in the function:  test_enableUpgradePath
        MapleProxyFactory.t.sol Variable factory is read 4 times in the function:  test_disableUpgradePath
        MapleProxyFactory.t.sol Variable factory is read 4 times in the function:  test_upgradeInstance
        MapleProxyFactory.t.sol Variable implementation1 is read 2 times in the function:  test_upgradeInstance
        MapleProxyFactory.t.sol Variable implementation2 is read 2 times in the function:  test_upgradeInstance


