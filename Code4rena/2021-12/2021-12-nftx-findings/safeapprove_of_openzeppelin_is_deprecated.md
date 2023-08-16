## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [safeApprove of openZeppelin is deprecated](https://github.com/code-423n4/2021-12-nftx-findings/issues/31) 

# Handle

robee


# Vulnerability details

        Deprecated safeApprove in NFTXMarketplaceZap.sol line 518: IERC20Upgradeable(vault).approve(address(sushiRouter), maxTokenIn); 
        Deprecated safeApprove in NFTXMarketplaceZap.sol line 537: IERC20Upgradeable(vault).approve(address(sushiRouter), maxTokenIn); 
        Deprecated safeApprove in NFTXSimpleFeeDistributor.sol line 158: IERC20Upgradeable(_vault).approve(_receiver.receiver, amountToSend); 
        Deprecated safeApprove in NFTXStakingZap.sol line 170: IERC20Upgradeable(address(IUniswapV2Router01(_sushiRouter).WETH())).approve(_sushiRouter, type(uint256).max); 
        Deprecated safeApprove in NFTXStakingZap.sol line 383: IERC20Upgradeable(vault).approve(address(sushiRouter), minTokenIn); 
        Deprecated safeApprove in NFTXStakingZap.sol line 397: IERC20Upgradeable(lpToken).approve(address(lpStaking), liquidity); 
        Deprecated safeApprove in PalmNFTXStakingZap.sol line 166: IERC20Upgradeable(address(_pairedToken)).approve(_sushiRouter, type(uint256).max); 
        Deprecated safeApprove in PalmNFTXStakingZap.sol line 298: IERC20Upgradeable(vault).approve(address(sushiRouter), minTokenIn); 
        Deprecated safeApprove in PalmNFTXStakingZap.sol line 312: IERC20Upgradeable(lpToken).approve(address(lpStaking), liquidity); 
        Deprecated safeApprove in NFTXFlashSwipe.sol line 56: IERC20Upgradeable(vault).approve(address(vault), allowance + count); 

