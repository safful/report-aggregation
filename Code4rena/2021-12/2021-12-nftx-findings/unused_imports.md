## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-12-nftx-findings/issues/19) 

# Handle

robee


# Vulnerability details

In the following files there are contract imports that aren't used. 
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore). 
The following is a full list of all unused imports, we went through the whole code to find it :) <solidity file> <line number> <actual import line>: 

        IPrevNftxContract.sol, line 4, import "./IERC165Upgradeable.sol";
        NFTXEligibilityManager.sol, line 5, import "./interface/INFTXVaultFactory.sol";
        NFTXInventoryStaking.sol, line 6, import "./interface/INFTXFeeDistributor.sol";
        NFTXInventoryStaking.sol, line 10, import "./token/IERC721Upgradeable.sol";
        NFTXInventoryStaking.sol, line 11, import "./token/IERC1155Upgradeable.sol";
        NFTXInventoryStaking.sol, line 16, import "./proxy/Initializable.sol";
        NFTXLPStaking.sol, line 5, import "./interface/INFTXFeeDistributor.sol";
        NFTXLPStaking.sol, line 12, import "./proxy/Initializable.sol";
        NFTXMarketplaceZap.sol, line 8, import "./interface/ITimelockRewardDistributionToken.sol";
        NFTXMarketplaceZap.sol, line 10, import "./testing/IERC721.sol";
        NFTXMarketplaceZap.sol, line 15, import "./util/OwnableUpgradeable.sol";
        NFTXSimpleFeeDistributor.sol, line 10, import "./util/SafeMathUpgradeable.sol";
        NFTXStakingZap.sol, line 9, import "./interface/ITimelockRewardDistributionToken.sol";
        NFTXStakingZap.sol, line 11, import "./testing/IERC721.sol";
        NFTXStakingZap.sol, line 16, import "./util/OwnableUpgradeable.sol";
        NFTXVaultFactoryUpgradeable.sol, line 5, import "./interface/INFTXLPStaking.sol";
        NFTXVaultFactoryUpgradeable.sol, line 7, import "./proxy/ClonesUpgradeable.sol";
        NFTXVaultUpgradeable.sol, line 8, import "./interface/INFTXLPStaking.sol";
        NFTXVaultUpgradeable.sol, line 10, import "./interface/IERC165Upgradeable.sol";
        PalmNFTXStakingZap.sol, line 15, import "../util/OwnableUpgradeable.sol";
        StakingTokenProvider.sol, line 7, import "./token/IERC20Upgradeable.sol";
        ERC20FlashMintUpgradeable.sol, line 4, import "../interface/IERC3156Upgradeable.sol";
        RewardDistributionTokenUpgradeable.sol, line 5, import "../interface/IRewardDistributionToken.sol";
        RewardDistributionTokenUpgradeable.sol, line 11, import "hardhat/console.sol";
        TimelockRewardDistributionTokenImpl.sol, line 5, import "../interface/IRewardDistributionToken.sol";
        NFTXFlashSwipe.sol, line 4, import "../interface/IERC3156Upgradeable.sol";
        PausableUpgradeable.sol, line 5, import "./SafeMathUpgradeable.sol";



