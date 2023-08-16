## Tags

- bug
- G (Gas Optimization)
- mStableYieldSource
- SwappableYieldSource
- sponsor confirmed

# [Declaring functions as `external` to save gas](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/55) 

# Handle

shw


# Vulnerability details

## Impact

In general, if not called by the contract itself, public functions can be declared as `external` to save gas.

## Proof of Concept

Referenced code:
[MStableYieldSource.sol#L61](https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L61)
[MStableYieldSource.sol#L69](https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L69)
[SwappableYieldSource.sol#L67](https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L67)
[SwappableYieldSource.sol#L98](https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L98)
[SwappableYieldSource.sol#L219](https://github.com/pooltogether/swappable-yield-source/blob/89cf66a3e3f8df24a082e1cd0a0e80d08953049c/contracts/SwappableYieldSource.sol#L219)

## Recommended Mitigation Steps

Change `public` to `external` in the referenced functions.

