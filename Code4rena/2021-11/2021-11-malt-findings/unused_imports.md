## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-11-malt-findings/issues/157) 

# Handle

robee


# Vulnerability details

In the following files, there are contract imports that aren't used. 
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore). 
The following is a full list of all unused imports, we went through the whole code to find it :) <solidity file> <line number> <actual import line>: 

        AuctionBurnReserveSkew.sol, line 4, import "@openzeppelin/contracts/access/AccessControl.sol";
        AuctionParticipant.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        AuctionParticipant.sol, line 3, import "@openzeppelin/upgrades/contracts/Initializable.sol";
        AuctionPool.sol, line 9, import "./interfaces/IAuction.sol";
        AuctionPool.sol, line 10, import "./interfaces/IBurnMintableERC20.sol";
        AuctionPool.sol, line 11, import "./interfaces/IDexHandler.sol";
        Bonding.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        ERC20VestedMine.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        ImpliedCollateralService.sol, line 12, import "./interfaces/IRewardThrottle.sol";
        LiquidityExtension.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        LiquidityExtension.sol, line 3, import "@openzeppelin/contracts/token/ERC20/SafeERC20.sol";
        MaltDataLab.sol, line 7, import "./interfaces/IStabilizerNode.sol";
        MaltDataLab.sol, line 9, import "./interfaces/IDAO.sol";
        MiningService.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        MiningService.sol, line 3, import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
        PoolTransferVerification.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        PoolTransferVerification.sol, line 6, import "./Permissions.sol";
        RewardDistributor.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        RewardDistributor.sol, line 11, import "hardhat/console.sol";
        RewardOverflowPool.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        RewardThrottle.sol, line 2, import "@openzeppelin/contracts/math/SafeMath.sol";
        RewardThrottle.sol, line 13, import "hardhat/console.sol";
        SwingTrader.sol, line 8, import "./interfaces/IAuction.sol";
        TransferService.sol, line 4, import "./interfaces/IMaltDataLab.sol";


