## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Simplify `sqrt()` can save gas](https://github.com/code-423n4/2021-10-mochi-findings/issues/115) 

# Handle

WatchPug


# Vulnerability details

The check of `x > 3` is unnecessary and most certainly adds more gas cost than it saves as the majority of use cases of this function will not be handling `x <= 3`.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-cssr/contracts/adapter/UniswapV2LPAdapter.sol#L106-L117

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-cssr/contracts/adapter/SushiswapV2LPAdapter.sol#L106-L117

```solidity=106
function sqrt(uint x) internal pure returns (uint y) {
    if (x > 3) {
        uint z = x / 2 + 1;
        y = x;
        while (z < y) {
            y = z;
            z = (x / z + z) / 2;
        }
    } else if (x != 0) {
        y = 1;
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

