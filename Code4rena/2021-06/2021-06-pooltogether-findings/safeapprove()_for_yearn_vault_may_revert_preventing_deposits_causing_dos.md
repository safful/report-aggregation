## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [safeApprove() for Yearn Vault may revert preventing deposits causing DoS](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/71) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The _depositInVault() function for Yearn yield source uses ERC20 safeApprove() from OpenZeppelin's SafeERC20 library to give maximum allowance to the Yearn Vault address if the current allowance is less than contract’s token balance.

However, the safeApprove function prevents changing an allowance between non-zero values to mitigate a possible front-running attack. It reverts if that is the case. Instead, the safeIncreaseAllowance and safeDecreaseAllowance functions should be used. Comment from the OZ library for this function: “// safeApprove should only be called when setting an initial allowance, // or when resetting it to zero. To increase and decrease it, use // 'safeIncreaseAllowance' and ‘safeDecreaseAllowance'"

Impact: If the existing allowance is non-zero (say, for e.g., previously the entire balance was not deposited due to vault balance limit resulting in the allowance being reduced but not made 0), then safeApprove() will revert causing the user’s token deposits to fail leading to denial-of-service. The condition predicate indicates that this scenario is possible.

## Proof of Concept

Reference: See similar Medium-severity finding M03 here: https://blog.openzeppelin.com/1inch-exchange-audit/

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/yield-source/YearnV2YieldSource.sol#L171-L173

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6842518b1b71fac9a21c7d94ec521992cff266b5/contracts/token/ERC20/utils/SafeERC20.sol#L44-L57


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use safeIncreaseAllowance() function instead of safeApprove().

