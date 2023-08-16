## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`SwapUtils.sol#getD()` Remove unnecessary variable and internal call can make the code simpler and save some gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/238) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L620-L623

```solidity=620
function getD(Swap storage self) internal view returns (uint256) {
    uint256 a = determineA(self, _xp(self));            // determine the correct A
    return getD(_xp(self), a);
}
```

`a` is unnecessary as it's being used only once. The result of `_xp(self)` can be cached to avoid calling it twice.

### Recommendation

Change to:

```solidity=620
function getD(Swap storage self) internal view returns (uint256) {
    uint256[] memory xp = _xp(self);
    return getD(xp, determineA(self, xp));
}
```

