## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Misleading error message](https://github.com/code-423n4/2021-11-malt-findings/issues/286) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L80-L86

```solidity=80
function emergencyWithdrawGAS(address payable destination)
    external 
    onlyRole(TIMELOCK_ROLE, "Only timelock can assign roles")
  {
    // Transfers the entire balance of the Gas token to destination
    destination.call{value: address(this).balance}('');
  }
```

The error message "Only timelock can assign roles" can be changed to "Only timelock can emergencyWithdrawGAS".

Other examples include:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Permissions.sol#L104-L110

```solidity=104
function partialWithdraw(address _token, address destination, uint256 amount)
    external 
    onlyRole(TIMELOCK_ROLE, "Only timelock can assign roles")
  {
    ERC20 token = ERC20(_token);
    token.safeTransfer(destination, amount);
  }
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/PoolTransferVerification.sol#L94-L94

```solidity=94
require(_pool != address(0), "Cannot have 0 lookback");
```

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/AuctionEscapeHatch.sol#L230-L230

```solidity=230
require(_period > 0, "Cannot have 0 lookback period");
```

