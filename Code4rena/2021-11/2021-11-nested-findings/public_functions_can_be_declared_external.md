## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Public functions can be declared external](https://github.com/code-423n4/2021-11-nested-findings/issues/72) 

# Handle

palina


# Vulnerability details

## Impact
Functions that are only called from outside the contract can be declared external instead of public since they are more gas-efficient.

## Proof of Concept
NestedBuybacker::setBurnPart(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/NestedBuybacker.sol#L81,
MixinOperatorResolver::rebuildCache(): https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/MixinOperatorResolver.sol#L29

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Change the 'public' visibility modifier into 'external'.

