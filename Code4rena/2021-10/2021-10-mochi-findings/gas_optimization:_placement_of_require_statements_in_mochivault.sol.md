## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization: Placement of require statements in MochiVault.sol](https://github.com/code-423n4/2021-10-mochi-findings/issues/27) 

# Handle

gzeon


# Vulnerability details

## Impact
Some of the require statements in MochiVault.sol can be placed earlier to reduce gas usage on revert

## Proof of Concept
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/vault/MochiVault.sol
L226-227: can be placed at the very top of the function to avoid the expensive cssr call
L237: can be placed before initialization of increasingDebt

## Tools Used
None

## Recommended Mitigation Steps
Relocate the said require statements

