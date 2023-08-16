## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Storage double reading. Could save SLOAD](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/8) 

# Handle

robee


# Vulnerability details

Reading a storage variable is gas costly (SLOAD). In cases of multiple read of a storage variable in the same scope, caching the first read (i.e saving as a local variable) can save gas and decrease the
 overall gas uses. The following is a list of functions and the storage variables that you read twice: 

        WJLP.sol Variable activePool is read 2 times in the function: unwrapFor
        WJLP.sol Variable stabilityPool is read 2 times in the function: unwrapFor
        Pool2Unipool.sol Variable periodFinish is read 2 times in the function: _notifyRewardAmount
        Pool2Unipool.sol Variable periodFinish is read 2 times in the function: _updatePeriodFinish
        Unipool.sol Variable periodFinish is read 2 times in the function: _notifyRewardAmount
        Unipool.sol Variable periodFinish is read 2 times in the function: _updatePeriodFinish
        PriceFeed.sol Variable lastGoodPrice is read 17 times in the function: fetchPrice
        PriceFeed.sol Variable lastGoodPrice is read 17 times in the function: fetchPrice_v
        EchidnaTester.sol Variable troveManager is read 9 times in the function: constructor
        EchidnaTester.sol Variable borrowerOperations is read 8 times in the function: constructor
        EchidnaTester.sol Variable activePool is read 6 times in the function: constructor
        EchidnaTester.sol Variable defaultPool is read 4 times in the function: constructor
        EchidnaTester.sol Variable stabilityPool is read 6 times in the function: constructor
        EchidnaTester.sol Variable gasPool is read 3 times in the function: constructor
        EchidnaTester.sol Variable troveManagerLiquidations is read 4 times in the function: constructor
        EchidnaTester.sol Variable troveManagerRedemptions is read 5 times in the function: constructor
        EchidnaTester.sol Variable yusdToken is read 5 times in the function: constructor
        EchidnaTester.sol Variable whitelist is read 4 times in the function: constructor
        EchidnaTester.sol Variable collSurplusPool is read 4 times in the function: constructor
        EchidnaTester.sol Variable priceFeedTestnet is read 3 times in the function: constructor
        EchidnaTester.sol Variable sortedTroves is read 4 times in the function: constructor
        EchidnaTester.sol Variable MCR is read 2 times in the function: constructor
        EchidnaTester.sol Variable CCR is read 2 times in the function: constructor
        ERC20.sol Variable totalSupply is read 2 times in the function: _mint
        TeamLockup.sol Variable totalClaimed is read 2 times in the function: claimYeti
        YETIToken.sol Variable _totalSupply is read 2 times in the function: constructor


