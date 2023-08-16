## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [unchecked arithmetics](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/51) 

# Handle

pauliax


# Vulnerability details

## Impact
You can save some gas by using the 'unchecked' keyword to avoid redundant arithmetic checks when an underflow/overflow cannot happen. For example, here:
  while (_prizeSplits.length > newPrizeSplitsLength) {
    uint256 _index = _prizeSplits.length - 1;
or here:
        require(_accountDetails.balance >= _amount, _revertMessage);
        ...
        accountDetails.balance = _accountDetails.balance - _amount;

## Recommended Mitigation Steps
Consider applying 'unchecked' keyword where overflows/underflows are not possible.

