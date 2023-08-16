## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`SwapUtils.sol#getYD()` Remove redundant code can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/233) 

# Handle

WatchPug


# Vulnerability details

`getYD()` already `require(tokenIndex < numTokens, "...")`, so the check in `getYDC()` is redundant.

Removing it will make the code simpler and save some gas.

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L477-L502

```solidity=477
function getYDC(
    Swap storage self,
    uint256 a,
    uint8 tokenIndex,
    uint256[] memory xp,
    uint256 d
) internal view returns (uint256) {
    uint256 numTokens = xp.length;
    require(tokenIndex < numTokens, "Token not found");

    // calculate y
    uint256 y = getYD(a, tokenIndex, xp, d);
    // ...
}
```

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L522-L557
```solidity=522
function getYD(
    uint256 a,
    uint8 tokenIndex,
    uint256[] memory xp,
    uint256 d
) internal pure returns (uint256) {
    uint256 numTokens = xp.length;
    require(tokenIndex < numTokens, "Token not found");
    // ...
}
```


### Recommendation

Remove the redundant code.

