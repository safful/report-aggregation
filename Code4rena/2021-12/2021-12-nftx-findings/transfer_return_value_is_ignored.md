## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [transfer return value is ignored](https://github.com/code-423n4/2021-12-nftx-findings/issues/40) 

# Handle

robee


# Vulnerability details

Need to use safeTransfer instead of transfer. As there are popular tokens, such as USDT that transfer/trasnferFrom method doesn’t return anything. The transfer return value has to be checked (as there are some other tokens that returns false instead revert), that means you must 
 1. Check the transfer return value
Another popular possibility is to add a whiteList.
Those are the appearances (solidity file, line number, actual line):

        NFTXStakingZap.sol, 401, IERC20Upgradeable(vault).transfer(to, minTokenIn-amountToken); 
        NFTXStakingZap.sol, 474, IERC20Upgradeable(token).transfer(msg.sender, IERC20Upgradeable(token).balanceOf(address(this))); 
        PalmNFTXStakingZap.sol, 190, pairedToken.transferFrom(msg.sender, address(this), wethIn); 
        PalmNFTXStakingZap.sol, 195, pairedToken.transfer(to, wethIn-amountEth); 
        PalmNFTXStakingZap.sol, 219, pairedToken.transferFrom(msg.sender, address(this), wethIn); 
        PalmNFTXStakingZap.sol, 224, pairedToken.transfer(to, wethIn-amountEth); 
        PalmNFTXStakingZap.sol, 316, IERC20Upgradeable(vault).transfer(to, minTokenIn-amountToken); 
        XTokenUpgradeable.sol, 54, baseToken.transfer(who, what); 
        NFTXFlashSwipe.sol, 51, IERC20Upgradeable(vault).transferFrom(msg.sender, address(this), mintFee + targetRedeemFee); 


