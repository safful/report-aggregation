## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Revert transaction if it is unable to change data](https://github.com/code-423n4/2021-11-malt-findings/issues/198) 

# Handle

xYrYuYx


# Vulnerability details

## Impact
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/TransferService.sol#L62
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/TransferService.sol#L78

In addVerifier and removeVerifier functions of TransferService.sol, it just returns instead of revert if it is unable to change data.
Revert transaction to avoid creating unnecessary transaction and save transaction cost.


## Tools Used
Manual

## Recommended Mitigation Steps
Revert transaction instead of return.

