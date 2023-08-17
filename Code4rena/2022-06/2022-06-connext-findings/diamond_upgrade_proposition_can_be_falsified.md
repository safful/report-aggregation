## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Diamond upgrade proposition can be falsified](https://github.com/code-423n4/2022-06-connext-findings/issues/241) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/facets/DiamondCutFacet.sol#L16-L29
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L94-L118
https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/LibDiamond.sol#L222-L240


# Vulnerability details

## Impact
Diamond is to be upgraded after a certain delay to give time to the community to verify changes made by the developers. If the proposition can be falsified, the contract admins can exploit the contract in any way of their choice.

## Proof of Concept
To determine the id of the proposal, only its facet changes are hashed, skipping two critical pieces of data - the `_init` and `_calldata`. During a diamond upgrade, devs can choose what code will be executed **by the contract using a delegatecall**. Thus, they can make the contract perform **any** actions of their choice.

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Add `_init` and `_calldata` to the proposition hash.


