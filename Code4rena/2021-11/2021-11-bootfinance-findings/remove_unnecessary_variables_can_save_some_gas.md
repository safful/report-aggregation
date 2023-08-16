## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove unnecessary variables can save some gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/223) 

# Handle

WatchPug


# Vulnerability details

`transferredDx` is unnecessary, it can be replaced with `dx`.

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1098-L1123

```solidity=1098{1119-1123}
function swap(
    Swap storage self,
    uint8 tokenIndexFrom,
    uint8 tokenIndexTo,
    uint256 dx,
    uint256 minDy
) external returns (uint256) {
    require(
        dx <= self.pooledTokens[tokenIndexFrom].balanceOf(msg.sender),
        "Cannot swap more than you own"
    );

    // Transfer tokens first to see if a fee was charged on transfer
    uint256 beforeBalance =
        self.pooledTokens[tokenIndexFrom].balanceOf(address(this));
    self.pooledTokens[tokenIndexFrom].safeTransferFrom(
        msg.sender,
        address(this),
        dx
    );

    // Use the actual transferred amount for AMM math
    uint256 transferredDx =
        self.pooledTokens[tokenIndexFrom].balanceOf(address(this)).sub(
            beforeBalance
        );
    // ...
```

### Recommendation

Change to:

```solidity
// Use the actual transferred amount for AMM math
uint256 dx =
    self.pooledTokens[tokenIndexFrom].balanceOf(address(this)).sub(
        beforeBalance
    );
// ...
```

