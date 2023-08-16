## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Assigning keccak operations to constant variables results in extra gas costs](https://github.com/code-423n4/2021-11-fairside-findings/issues/3) 

# Handle

TomFrench


# Vulnerability details

## Impact

Increased gas costs

## Proof of Concept

In a number of places a `keccak("string")` expression is assigned to a `constant` variable. Due to how `constant` variables are implemented this results in the hash being recomputed each time that the variable is used, spending the gas necessary to perform this action.

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/dao/FairSideDAO.sol#L43-L49

If these variables were to be `immutable` this hash is calculated once at deploy time and then the result is saved to be used directly at runtime rather than recalculating, saving the cost of hashing.

See: https://github.com/ethereum/solidity/issues/9232

## Recommended Mitigation Steps

Change all `constant` hashes to be `immutable`

