## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Fixed18 conversions don't work for all values](https://github.com/code-423n4/2021-12-perennial-findings/issues/54) 

# Handle

cmichel


# Vulnerability details

Certain functions in the `Fixed18` contract perform multiplications by `ONE` or `NEG_ONE` before diving by it again which leads to issues that these functions revert for all values `> MAX_UINT256 / ONE`, but they should not.

```solidity
function from(int256 s, UFixed18 m) internal pure returns (Fixed18) {
    if (s > 0) return from(m);
    // @audit cannot convert large values because (m * NEG_ONE) might overflow
    if (s < 0) return mul(from(m), NEG_ONE);
    return ZERO;
}

function abs(Fixed18 a) internal pure returns (UFixed18) {
    // @audit cannot get abs value if multiplication of a * -1e18 /1e18 overflows. why not unwrap => unary minus
    return sign(a) == -1 ? UFixed18Lib.from(mul(a, NEG_ONE)) : UFixed18Lib.from(a);
}
```

## Recommendation
Change the implementation to not perform the useless `* 1e18 / 1e18` computations to cover the entire input range.
Consider using a typecast `int256(UFixed18.unwrap(m))` after checking the range instead of doing `* NEG_ONE / 1e18`


