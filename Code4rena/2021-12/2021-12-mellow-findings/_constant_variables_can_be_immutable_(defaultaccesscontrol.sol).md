## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [ Constant variables can be immutable (DefaultAccessControl.sol)](https://github.com/code-423n4/2021-12-mellow-findings/issues/122) 

# Handle

ye0lde


# Vulnerability details

## Impact

Changing the variables from constant to immutable will reduce keccak operations and save gas.

## Proof of Concept

The variables that can be changed from `constant` to `immutable` are here:
https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/DefaultAccessControl.sol#L11-L12

A previous finding with additional explanation and a pointer to the Ethereum/solidity issue is here:
https://github.com/code-423n4/2021-10-slingshot-findings/issues/3

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Change the constant variables to immutable.

