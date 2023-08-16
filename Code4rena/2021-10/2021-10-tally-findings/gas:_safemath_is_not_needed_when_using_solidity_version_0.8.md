## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: SafeMath is not needed when using Solidity version 0.8](https://github.com/code-423n4/2021-10-tally-findings/issues/42) 

# Handle

cmichel


# Vulnerability details

The `Swap` contract uses Solidity version 0.8 which already implements overflow checks by default.
At the same time, it uses the `SafeMath` library which is more gas expensive than the 0.8 overflow checks.

It should just use the built-in checks and remove `SafeMath` from the dependencies:

```solidity
// @audit can just normal arithmetic here
uint256 toTransfer = SWAP_FEE_DIVISOR.sub(swapFee).mul(boughtERC20Amount).div(SWAP_FEE_DIVISOR);

// uint256 toTransfer = (SWAP_FEE_DIVISOR - swapFee) * boughtERC20Amount / SWAP_FEE_DIVISOR;

// same with many other computations
```

