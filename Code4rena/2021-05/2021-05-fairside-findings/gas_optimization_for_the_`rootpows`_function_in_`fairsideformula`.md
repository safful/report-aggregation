## Tags

- bug
- question
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Gas optimization for the `rootPows` function in `FairSideFormula`](https://github.com/code-423n4/2021-05-fairside-findings/issues/71) 

# Handle

shw


# Vulnerability details

## Impact

Gas optimization is possible for the current `rootPows` implementation.

## Proof of Concept

The original implementation of `rootPows` requires 4 `mul` and 2 `sqrt`:

```solidity
function rootPows(bytes16 x) private pure returns (bytes16, bytes16) {
    // fourth root
    x = x.sqrt().sqrt();
    // to the power of 3
    x = _pow3(x);
    // we offset the root on the second arg
    return (x, x.mul(x));
}
```

However, the calculation process can be simplified to be more gas-efficient than the original with only 1 `mul` and 2 `sqrt` requried:

```solidity
function rootPows(bytes16 x) private pure returns (bytes16, bytes16) {
    bytes16 x1_2 = x.sqrt();
    bytes16 x3_2 = x.mul(x1_2);
    bytes16 x3_4 = x3_2.sqrt();
    return (x3_4, x3_2);
}
```

Referenced code:
[FairSideFormula.sol#L67-L75](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/FairSideFormula.sol#L67-L75)

## Recommended Mitigation Steps

To save gas, change the implementation of `rootPows` as mentioned above.

