## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`SafeCast.sol#toUint128()` Validation of input value can be done earlier to save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/173) 

# Handle

WatchPug


# Vulnerability details

Check input value earlier can avoid unnecessary code execution when this check failed.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/SafeCast.sol#L13-L15

```solidity
function toUint128(uint256 x) internal pure returns (uint128 y) {
    require((y = uint128(x)) == x);
}
```

See: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/4961a51cc736c7d4aa9bd2e11e4cbbaff73efee9/contracts/utils/math/SafeCast.sol#L48

### Recommendation

Change to:

```solidity
function toUint128(uint256 value) internal pure returns (uint128) {
    require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
    return uint128(value);
}
```

`SafeCast.sol#toUint112()` got the similar issue.

