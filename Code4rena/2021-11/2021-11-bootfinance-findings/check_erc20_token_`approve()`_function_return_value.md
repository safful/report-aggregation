## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Check ERC20 token `approve()` function return value](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/109) 

# Handle

Ruhum


# Vulnerability details

## Impact
With version 4.x of the ERC20 token, the `approve()` function returns a boolean indicating whether it was successful or not.

https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-

Best practice is to either check the return value or use `safeApprove()` / `safeIncreaseAllowance()` which will revert if the operation was unsuccessful

https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeApprove-contract-IERC20-address-uint256-

## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L499

https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L80

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
use `safeApprove()` or `safeIncreaseAllowance()`

