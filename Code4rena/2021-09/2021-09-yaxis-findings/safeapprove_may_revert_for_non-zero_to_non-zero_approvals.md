## Tags

- bug
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed

# [safeApprove may revert for non-zero to non-zero approvals](https://github.com/code-423n4/2021-09-yaxis-findings/issues/63) 

# Handle

0xRajeev


# Vulnerability details

## Impact 
OpenZeppelin’s safeApprove reverts for non-0 to non-0 approvals. This is considered in a few places where safeApprove is performed twice with the first one for 0 and then for the desired allowance. However, there are uses of safeApprove where it is called only once for the desired allowance but it is not clear that the current allowance is guaranteed to be zero. In such cases, safeApprove may revert.

## Proof of Concept

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6241995ad323952e38f8d405103ed994a2dcde8e/contracts/token/ERC20/utils/SafeERC20.sol#L49-L55

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/converters/StablesConverter.sol#L78

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/strategies/BaseStrategy.sol#L88

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Approve to 0 first while using safeApprove or use increaseAllowance instead.

