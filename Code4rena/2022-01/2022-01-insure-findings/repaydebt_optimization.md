## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [repayDebt optimization](https://github.com/code-423n4/2022-01-insure-findings/issues/307) 

# Handle

pauliax


# Vulnerability details

## Impact
function repayDebt could be refactored to reduce deployment and operational costs
from this:
```solidity
  uint256 _debt = debts[_target];
  if (_debt >= _amount) {
      debts[_target] -= _amount;
      totalDebt -= _amount;
      IERC20(token).safeTransferFrom(msg.sender, address(this), _amount);
  } else {
      debts[_target] = 0;
      totalDebt -= _debt;
      IERC20(token).safeTransferFrom(msg.sender, address(this), _debt);
  }
```
to this:
```solidity
  uint256 _debt = debts[_target];
  if (_debt > _amount) {
      debts[_target] = _debt - _amount;
  } else {
      debts[_target] = 0;
      _amount = _debt;
  }
  totalDebt -= _amount;
  IERC20(token).safeTransferFrom(msg.sender, address(this), _amount);
```

