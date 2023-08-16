## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [ERC20 return values not checked](https://github.com/code-423n4/2021-11-unlock-findings/issues/157) 

# Handle

cmichel


# Vulnerability details

The `ERC20.transfer()` and `ERC20.transferFrom()` functions return a boolean value indicating success. This parameter should checked for success.

See `Unlock.recordKeyPurchase` which performs ERC20 transfers without checking for the return value.

## Impact
As the trusted `udt` token is used which supposedly reverts on failed transfers, not checking the return value does not lead to any security issues.
We still recommend checking it to abide by the EIP20 standard.

## Recommended Mitigation Steps
Consider using `require(IMintableERC20(udt).transfer(_referrer, tokensToDistribute - devReward), "transfer failed")` instead.

