## Tags

- bug
- G (Gas Optimization)
- SwappableYieldSource
- sponsor confirmed

# [Redundant zero-address check](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/33) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The zero-address check on owner is present even in transferOwnership() which makes it redundant.

## Proof of Concept

https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L110


https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/081776bf5fae2122bfda8a86d5369496adfdf959/contracts/access/OwnableUpgradeable.sol#L68

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove explicit check to rely on the one in transferOwnership().

