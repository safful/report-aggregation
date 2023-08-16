## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unnecessary require checker](https://github.com/code-423n4/2021-10-covalent-findings/issues/3) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
In `disableValidator` function, validatorId checker is not required, or it is good to change require order for better contract.

## Proof of Concept
If `validatorId` is higher than `validatorsN`, it means, that validator is not initialized, so `validator._address` is always `address(0)`. so it will revert in Line 358.
It means that Line 359 cannot be executed at all.

## Tools Used

## Recommended Mitigation Steps
Move Line 359 (https://github.com/code-423n4/2021-10-covalent/blob/main/contracts/DelegatedStaking.sol#L359) at the top of function body, before get validator storage variable.
This is good to track correct issue.

Or 

You can remove that line.
So if validatorId is invalid, the error message will be `Caller is not the owner or the validator`, because validator._address = address(0) which cannot be caller.

