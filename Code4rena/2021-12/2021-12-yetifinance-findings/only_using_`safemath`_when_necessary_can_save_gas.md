## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Only using `SafeMath` when necessary can save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/281) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using SafeMath will cost more gas.

For example:

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/BorrowerOperations.sol#L791-L795

```solidity=791
if (_debtChange > _variableYUSDFee) { // if debt decrease, and greater than variable fee, decrease 
    newDebt = _troveManager.decreaseTroveDebt(_borrower, _debtChange.sub(_variableYUSDFee));
} else { // otherwise increase by opposite subtraction
    newDebt = _troveManager.increaseTroveDebt(_borrower, _variableYUSDFee.sub(_debtChange));
}
```

`_debtChange - _variableYUSDFee` at L792 and `_variableYUSDFee - _debtChange` at L794 will never underflow.


https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/YUSDToken.sol#L240-L241

```solidity=240
_totalSupply = _totalSupply.add(amount);
_balances[account] = _balances[account].add(amount);
```

`_balances[account] + amount` will not overflow if `_totalSupply.add(amount)` dose not overflow. 


https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/YUSDToken.sol#L248-L249

```solidity=248
_balances[account] = _balances[account].sub(amount, "ERC20: burn amount exceeds balance");
_totalSupply = _totalSupply.sub(amount);
```

`_totalSupply - amount` will not underflow if `_balances[account].sub(amount)` dose not underflow. 

