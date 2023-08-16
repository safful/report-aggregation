## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`XDEFIDistribution.sol#relock()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2022-01-xdefi-findings/issues/123) 

# Handle

WatchPug


# Vulnerability details

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L120-L125

```solidity=120
uint256 withdrawAmount = amountUnlocked_ - lockAmount_;

if (withdrawAmount != uint256(0)) {
    // Send the excess XDEFI to the destination, if needed.
    SafeERC20.safeTransfer(IERC20(XDEFI), destination_, withdrawAmount);
}
```

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L175-L180

```solidity=175
uint256 withdrawAmount = amountUnlocked_ - lockAmount_;

if (withdrawAmount != uint256(0)) {
    // Send the excess XDEFI to the destination, if needed.
    SafeERC20.safeTransfer(IERC20(XDEFI), destination_, withdrawAmount);
}
```
### Recommendation

Change to:

```solidity
if (amountUnlocked_ > lockAmount_) {
    SafeERC20.safeTransfer(IERC20(XDEFI), destination_, amountUnlocked_ - lockAmount_);
}
```

- Removed a local variable: `withdrawAmount`;
- Only do the arithmetic when needed: `amountUnlocked_ - lockAmount_`.

