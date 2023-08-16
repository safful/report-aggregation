## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Only using `SafeMath` when necessary can save gas](https://github.com/code-423n4/2021-11-yaxis-findings/issues/41) 

# Handle

WatchPug


# Vulnerability details

or the arithmetic operations that will never over/underflow, using SafeMath will cost more gas.

For example:

https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/Alchemist.sol#L623-L637

```solidity=623
if (_totalCredit < _amount) {
    uint256 _remainingAmount = _amount.sub(_totalCredit);
    // ...
}
```

`_amount - _totalCredit` will never underflow.

### Recommendation

Change to:

```solidity=623
if (_totalCredit < _amount) {
    uint256 _remainingAmount = _amount - _totalCredit;
    // ...
}
```

