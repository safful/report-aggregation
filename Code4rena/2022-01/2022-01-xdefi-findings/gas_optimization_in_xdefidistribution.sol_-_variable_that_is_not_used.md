## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in XDEFIDistribution.sol - variable that is not used](https://github.com/code-423n4/2022-01-xdefi-findings/issues/120) 

# Handle

OriDabush


# Vulnerability details

## XDEFIDistribution.sol line 332 
The "currentDistributableXDEFI" variable is not used (can use distributableXDEFI instead).
```sol
// function before:
function _updateXDEFIBalance() internal returns (int256 newFundsTokenBalance_) {
    uint256 previousDistributableXDEFI = distributableXDEFI;
    uint256 currentDistributableXDEFI = distributableXDEFI = IERC20(XDEFI).balanceOf(address(this)) - totalDepositedXDEFI;
    return _toInt256Safe(currentDistributableXDEFI) - _toInt256Safe(previousDistributableXDEFI);
}

// function after:
function _updateXDEFIBalance() internal returns (int256 newFundsTokenBalance_) {
    uint256 previousDistributableXDEFI = distributableXDEFI;
    distributableXDEFI = IERC20(XDEFI).balanceOf(address(this)) - totalDepositedXDEFI;
    return _toInt256Safe(distributableXDEFI) - _toInt256Safe(previousDistributableXDEFI);
}
```

