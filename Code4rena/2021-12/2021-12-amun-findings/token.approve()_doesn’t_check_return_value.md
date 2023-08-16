## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [token.approve() doesn’t check return value](https://github.com/code-423n4/2021-12-amun-findings/issues/294) 

# Handle

sirhashalot


# Vulnerability details

## Impact

Multiple files within the contracts/basket/contracts/singleJoinExit/ directory call `token.approve()` for an ERC20 token, but these calls do not verify whether the `approve()` call failed. Some ERC20 tokens do not revert if an approval fails, and because the return value is not checked, the contract would not be aware of this failure, potentially causing malfunctions in later operations. Using a function from SafeERC20 that checks the return value would mitigate this edge case.

## Proof of Concept

token.approve() is found in several locations:
- singleJoinExit/SingleTokenJoin.sol [line 43](https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/SingleTokenJoin.sol#L43)
- singleJoinExit/SingleNativeTokenExitV2.sol [line 55](https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/SingleNativeTokenExitV2.sol#L55)
- singleJoinExit/SingleNativeTokenExit.sol [line 44](https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/SingleNativeTokenExit.sol#L44)
- singleJoinExit/SingleTokenJoinV2.sol [line 53](https://github.com/code-423n4/2021-12-amun/blob/98f6e2ff91f5fcebc0489f5871183566feaec307/contracts/basket/contracts/singleJoinExit/SingleTokenJoinV2.sol#L53)

## Tools Used

Manual analysis

## Recommended Mitigation Steps

While the OpenZeppelin SafeERC20 `safeApprove()` function could be used to revert on approve failures unlike the standard `approve()`, the `safeApprove()` function [is deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/5ac4d93ae318cdf6c8a1125dff3a439a2aeabb63/contracts/token/ERC20/utils/SafeERC20.sol#L39-L44) and instead OpenZeppelin recommends either `safeIncreaseAllowance()` or `safeDecreaseAllowance()`. Because uint256(-1) should be an increase, replace each instance of `token.approve(spender, uint256(-1))` with `token.safeIncreaseAllowance(spender, uint256(-1))`.

