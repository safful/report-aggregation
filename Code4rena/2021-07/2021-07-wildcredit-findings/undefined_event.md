## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Undefined Event](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/1) 

# Handle

defsec


# Vulnerability details

## Impact

Without Event, it is difficult to identify in real-time whether correct values are recorded on
the blockchain. In this case, it becomes problematic to determine whether the
corresponding value has been changed in the application and whether the corresponding
function has been called. setMinBorrowUSD function is missing event.

## Proof of Concept

1. Go to following the function [setMinBorrowUSD Function](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/Controller.sol#L137)
2. There is missing event definition on the function. 

## Tools Used

None

## Recommended Mitigation Steps

Add Event corresponding to the change occurring in the function.

<code>Add `emit NewMinBorrowUSD(_value);`</code>

