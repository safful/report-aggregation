## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Missing checks for non-zero transfer value calls](https://github.com/code-423n4/2022-01-notional-findings/issues/94) 

# Handle

Dravee


# Vulnerability details

## Impact  
Checking non-zero transfer values can avoid an external call to save gas.  
  
## Proof of Concept  
Instances missing a non-zero check on amount transfered:  
```  
contracts\sNOTE.sol:142:        BALANCER_POOL_TOKEN.safeTransferFrom(msg.sender, address(this), bptAmount);
contracts\sNOTE.sol:150:        NOTE.safeTransferFrom(msg.sender, address(this), noteAmount);
contracts\sNOTE.sol:178:        WETH.safeTransferFrom(msg.sender, address(this), wethAmount);
contracts\sNOTE.sol:251:        BALANCER_POOL_TOKEN.safeTransfer(msg.sender, bptToRedeem);
contracts\TreasuryAction.sol:113:        COMP.safeTransfer(treasuryManagerContract, amountClaimed);
contracts\TreasuryAction.sol:140:        IERC20(underlyingAddress).safeTransfer(treasuryManagerContract, redeemedExternalUnderlying);
contracts\TreasuryManager.sol:108:        IERC20(token).safeTransfer(owner, amount);
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Check if transfer amount > 0.


