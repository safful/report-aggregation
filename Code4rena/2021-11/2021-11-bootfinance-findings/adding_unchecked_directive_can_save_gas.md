## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/261) 

# Handle

defsec


# Vulnerability details

## Impact

Using the unchecked keyword to avoid redundant arithmetic underflow/overflow checks to save gas when an underflow/overflow cannot happen. E.g. 'unchecked' can be applied in the following lines of code since there are require statements before to ensure the arithmetic operations would not cause an integer underflow or overflow.

## Proof of Concept

1. Review the all contracts and add unchecked keyword where overflow is not possible.

"https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L579"

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider applying 'unchecked' keyword where overflows/underflows are not possible.



