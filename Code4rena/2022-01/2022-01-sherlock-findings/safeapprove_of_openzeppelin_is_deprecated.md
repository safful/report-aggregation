## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [safeApprove of openZeppelin is deprecated](https://github.com/code-423n4/2022-01-sherlock-findings/issues/11) 

# Handle

robee


# Vulnerability details

You use safeApprove of openZeppelin although it's deprecated. 
(see https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38)
You should change it to increase/decrease Allowance as OpenZeppilin says.
This appears in the following locations in the code base:

        Deprecated safeApprove in AaveV2Strategy.sol line 70: want.safeApprove(address(lp), type(uint256).max); 
        Deprecated safeApprove in SherlockClaimManager.sol line 421: TOKEN.safeApprove(address(UMA), _amount); 
        Deprecated safeApprove in SherlockClaimManager.sol line 464: TOKEN.safeApprove(address(UMA), 0); 
        Deprecated safeApprove in SherBuy.sol line 99: usdc.approve(address(sherlockPosition), type(uint256).max); 
        Deprecated safeApprove in SherBuy.sol line 169: sher.approve(address(sherClaim), sherAmount); 


