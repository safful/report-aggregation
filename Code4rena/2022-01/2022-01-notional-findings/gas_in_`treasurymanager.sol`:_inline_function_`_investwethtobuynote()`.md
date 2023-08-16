## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas in `TreasuryManager.sol`: Inline function `_investWETHToBuyNOTE()`](https://github.com/code-423n4/2022-01-notional-findings/issues/129) 

# Handle

Dravee


# Vulnerability details

Here's the only `_investWETHToBuyNOTE()` call:
```
File: TreasuryManager.sol
140:     function investWETHToBuyNOTE(uint256 wethAmount) external onlyManager {
141:         _investWETHToBuyNOTE(wethAmount);
142:     }
```

While I can understand why some functions have the same style, these other functions are as such because they are calling an inherited function, as an example for `setSlippageLimit()`, which really is calling an inherited function from `EIP1271Wallet.sol`:
```
File: TreasuryManager.sol
89:     function setSlippageLimit(address tokenAddress, uint256 slippageLimit)
90:         external
91:         onlyOwner
92:     {
93:         _setSlippageLimit(tokenAddress, slippageLimit);
94:     }
```

However, for `_investWETHToBuyNOTE()`, this style doesn't hold. 

## Recommended Mitigation Steps
All the logic from `_investWETHToBuyNOTE()` should be inlined in `investWETHToBuyNOTE()` to save gas.

