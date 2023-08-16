## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Unnecessary redundant check for basisPoints](https://github.com/code-423n4/2021-07-spartan-findings/issues/129) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The threshold check for basisPoints while a required part of input validation is an unnecessary redundant check because calcPart() does a similar upper bound check and the lower bound check on 0 is only an optimization.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L95

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Utils.sol#L65


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove redundant check to save gas and improve readability/maintainability.

