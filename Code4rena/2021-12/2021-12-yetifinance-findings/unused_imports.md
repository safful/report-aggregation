## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/2) 

# Handle

robee


# Vulnerability details

In the following files there are contract imports that aren't used. 
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore). 
The following is a full list of all unused imports, we went through the whole code to find it :) <solidity file> <line number> <actual import line>: 

        ActivePool.sol, line 6, import "./Interfaces/IStabilityPool.sol";
        ActivePool.sol, line 7, import "./Interfaces/IDefaultPool.sol";
        ActivePool.sol, line 13, import "./Dependencies/LiquityBase.sol";
        ERC20_8.sol, line 3, import "./Interfaces/IERC20_8.sol";
        IMasterChefJoeV2.sol, line 1, import "./IERC20_8.sol";
        IRewarder.sol, line 3, import "./IERC20_8.sol";
        LiquityBase.sol, line 8, import "../Interfaces/IWhitelist.sol";
        TroveManagerBase.sol, line 12, import "../Interfaces/IWhitelist.sol";
        Whitelist.sol, line 13, import "../Interfaces/IERC20.sol";
        Whitelist.sol, line 14, import "./LiquityMath.sol";
        YetiCustomBase.sol, line 6, import "../Interfaces/IERC20.sol";
        ICollSurplusPool.sol, line 4, import "../Dependencies/YetiCustomBase.sol";
        ILiquityBase.sol, line 4, import "./IPriceFeed.sol";
        MockAggregator.sol, line 5, import "hardhat/console.sol";
        NonPayable.sol, line 4, //import "hardhat/console.sol";
        SortedTrovesTester.sol, line 4, import "../Interfaces/ISortedTroves.sol";
        TroveManagerLiquidations.sol, line 6, import "hardhat/console.sol";
        sYETIToken.sol, line 9, import "./BoringCrypto/BoringBatchable.sol";

