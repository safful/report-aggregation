## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`SquareRoot#sqrtUp()` Wrong implementation](https://github.com/code-423n4/2022-01-timeswap-findings/issues/176) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/SquareRoot.sol#L19-L22

```solidity
function sqrtUp(uint256 y) internal pure returns (uint256 z) {
    z = sqrt(y);
    if (z % y > 0) z++;
}
```

For example, when `y = 9`:
-   At L20, z = sqrt(9) = 3
-   At L21, z % y = 3 % 9 = 3, so that `z % y > 0` is true, therefore, `z++`, z is 4


Expected Results: sqrtUp(9) = 4

Actual Results: sqrtUp(9) = 3

### Recommendation

Change to:

```solidity
function sqrtUp(uint256 y) internal pure returns (uint256 z) {
    z = sqrt(y);
    if (z * z < y) ++z;
}
```
or

```solidity
function sqrtUp(uint256 y) internal pure returns (uint256 z) {
    z = sqrt(y);
    if (y % z != 0) ++z;
}
```

