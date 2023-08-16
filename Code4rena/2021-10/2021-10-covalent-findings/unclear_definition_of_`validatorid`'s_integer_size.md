## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Unclear definition of `validatorId`'s integer size](https://github.com/code-423n4/2021-10-covalent-findings/issues/68) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
The mapping `validators` is defined with `uint` (alias for `uint256`) as key 
type.

In the functions receiving the `validatorId` as parameter however, the 
`validatorId` is defined as `uint128`.

See lines [166](https://github.com/code-423n4/2021-10-covalent/blob/main/contracts/DelegatedStaking.sol#L166), [214](https://github.com/code-423n4/2021-10-covalent/blob/main/contracts/DelegatedStaking.sol#L214).

The suggestion is to be consistent with the integer size.

