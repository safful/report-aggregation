## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Unchecked return value for `ERC20.approve` call](https://github.com/code-423n4/2021-12-maple-findings/issues/52) 

# Handle

WatchPug


# Vulnerability details

There are a few functions across the codebase that will perform an ERC20.approve() call but does not check the success return value. Some tokens do not revert if the approval failed but return false instead.

Instances include:

https://github.com/maple-labs/liquidations/blob/bb09e17b1fac1126ce7734e58c3133be06162590/contracts/SushiswapStrategy.sol#L55-L55
```solidity=55
ERC20Helper.approve(collateralAsset_, ROUTER, swapAmount_);
```

https://github.com/maple-labs/liquidations/blob/bb09e17b1fac1126ce7734e58c3133be06162590/contracts/UniswapV2Strategy.sol#L55-L55
```solidity=55
ERC20Helper.approve(collateralAsset_, ROUTER, swapAmount_);
```

It is usually good to add a require-statement that checks the return value or to use something like `safeApprove`; unless one is sure the given token reverts in case of a failure.

