## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Simplify `SquareRoot#sqrt()` can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/174) 

# Handle

WatchPug


# Vulnerability details

The check of `y > 3` is unnecessary and most certainly adds more gas cost than it saves as the majority of use cases of this function will not be handling `y <= 3`.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/SquareRoot.sol#L6-L17

```solidity
function sqrt(uint256 y) internal pure returns (uint256 z) {
    if (y > 3) {
        z = y;
        uint256 x = y / 2 + 1;
        while (x < z) {
            z = x;
            x = (y / x + x) / 2;
        }
    } else if (y != 0) {
        z = 1;
    }
}
```

### Recommendation

Change to:

```solidity
function sqrt(uint x) public pure returns (uint y) {
    uint z = (x + 1) / 2;
    y = x;
    while (z < y) {
        y = z;
        z = (x / z + z) / 2;
    }
}
```

Or use:

https://github.com/Rari-Capital/solmate/blob/dd13c61b5f9cb5c539a7e356ba94a6c2979e9eb9/src/utils/FixedPointMathLib.sol#L150-L205


