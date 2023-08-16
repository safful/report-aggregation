## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`XDEFIDistribution.sol#_updateXDEFIBalance()` Avoiding unnecessary storage writes can save gas](https://github.com/code-423n4/2022-01-xdefi-findings/issues/151) 

# Handle

WatchPug


# Vulnerability details

Storage writes (`SSTORE`) to `distributableXDEFI` may not be needed when `previousDistributableXDEFI == currentDistributableXDEFI`, therefore the code can be reorganized to save gas from unnecessary storage writes.

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L330-L336

```solidity
function _updateXDEFIBalance() internal returns (int256 newFundsTokenBalance_) {
    uint256 previousDistributableXDEFI = distributableXDEFI;
    uint256 currentDistributableXDEFI = distributableXDEFI = IERC20(XDEFI).balanceOf(address(this)) - totalDepositedXDEFI;

    return _toInt256Safe(currentDistributableXDEFI) - _toInt256Safe(previousDistributableXDEFI);
}
```

### Recommendation

Change to:

```solidity
function _updateXDEFIBalance() internal returns (int256 newFundsTokenBalance_) {
    uint256 previousDistributableXDEFI = distributableXDEFI;
    uint256 currentDistributableXDEFI = IERC20(XDEFI).balanceOf(address(this)) - totalDepositedXDEFI;

    newFundsTokenBalance_ = _toInt256Safe(currentDistributableXDEFI) - _toInt256Safe(previousDistributableXDEFI);
    if (newFundsTokenBalance_ != 0) {
        distributableXDEFI = currentDistributableXDEFI;
    }
}
```

