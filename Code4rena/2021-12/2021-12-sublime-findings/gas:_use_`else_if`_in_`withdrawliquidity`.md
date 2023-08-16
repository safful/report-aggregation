## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Use `else if` in `withdrawLiquidity`](https://github.com/code-423n4/2021-12-sublime-findings/issues/148) 

# Handle

cmichel


# Vulnerability details

The `if` conditions in `Pool.withdrawLiquidity` are distinct conditions on the pool status.
Therefore, `else if` is semantically equivalent but more gas efficient.

```solidity
if (_loanStatus == LoanStatus.DEFAULTED || _loanStatus == LoanStatus.TERMINATED) {
    uint256 _totalAsset;
    if (poolConstants.borrowAsset != address(0)) {
        _totalAsset = IERC20(poolConstants.borrowAsset).balanceOf(address(this));
    } else {
        _totalAsset = address(this).balance;
    }
    //assuming their will be no tokens in pool in any case except liquidation (to be checked) or we should store the amount in liquidate()
    _toTransfer = _toTransfer.mul(_totalAsset).div(totalSupply());
}
// @audit gas: use else if, status fields are distinct, only one of the branches is (if ever) executed anyway
if (_loanStatus == LoanStatus.CANCELLED) {
    _toTransfer = _toTransfer.add(_toTransfer.mul(poolVariables.penaltyLiquidityAmount).div(totalSupply()));
}

if (_loanStatus == LoanStatus.CLOSED) {
    //transfer repayment
    _withdrawRepayment(msg.sender);
}
```


