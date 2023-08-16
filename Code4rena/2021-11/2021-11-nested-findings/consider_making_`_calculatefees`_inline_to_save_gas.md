## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Consider making `_calculateFees` inline to save gas](https://github.com/code-423n4/2021-11-nested-findings/issues/167) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/NestedFactory.sol#L553-L559

```solidity=553
/// @dev Calculate the fees for a specific user and amount (1%)
/// @param _user The user address
/// @param _amount The amount
/// @return The fees amount
function _calculateFees(address _user, uint256 _amount) private view returns (uint256) {
    return _amount / 100;
}
```

The function `_calculateFees()` is a rather simple function, replacing it with inline expression `_amount / 100` can save some gas.

