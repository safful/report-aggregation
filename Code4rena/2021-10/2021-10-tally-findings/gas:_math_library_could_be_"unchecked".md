## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Math library could be "unchecked"](https://github.com/code-423n4/2021-10-tally-findings/issues/43) 

# Handle

cmichel


# Vulnerability details

The `Math` library uses Solidity version 0.8 which comes with built-in overflow checks which cost gas.

The code already checks for underflows (`a > b` before doing the division), and therefore the built-in checks can be disabled everywhere for improved gas cost.

```solidity
pragma solidity ^0.8.0;

library Math {
    function subOrZero(uint256 a, uint256 b) internal pure returns (uint256) {
      unchecked {
        return a > b ? a - b : 0;
      }
    }

    function subOrZero(uint128 a, uint128 b) internal pure returns (uint128) {
      unchecked {
        return a > b ? a - b : 0;
      }
    }

    function subOrZero(uint64 a, uint64 b) internal pure returns (uint64) {
      unchecked {
        return a > b ? a - b : 0;
      }
    }

    function subOrZero(uint32 a, uint32 b) internal pure returns (uint32) {
      unchecked {
        return a > b ? a - b : 0;
      }
    }
}
```

