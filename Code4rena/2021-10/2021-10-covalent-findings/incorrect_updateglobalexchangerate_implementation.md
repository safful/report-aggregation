## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor acknowledged
- sponsor confirmed

# [Incorrect updateGlobalExchangeRate implementation](https://github.com/code-423n4/2021-10-covalent-findings/issues/17) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
UpdateGlobalExchangeRate has incorrect implementation when `totalGlobalShares` is zero.

If any user didn't start stake, `totalGlobalShares` is 0, and every stake it will increase.
but there is possibility that `totalGlobalShares` can be 0 amount later by unstake or disable validator.

## Proof of Concept
https://github.com/xYrYuYx/C4-2021-10-covalent/blob/main/test/c4-tests/C4_issues.js#L76
This is my test case to proof this issue.

In my test case, I disabled validator to make `totalGlobalShares` to zero.
And in this case, some reward amount will be forever locked in the contract.
After disable validator, I mined 10 blocks, and 4 more blocks mined due to other function calls,
So total 14 CQT is forever locked in the contract.


## Tools Used
Hardhat test

## Recommended Mitigation Steps
Please think again when `totalGlobalShares` is zero.

