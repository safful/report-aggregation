## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache and read storage variables from the stack can save gas](https://github.com/code-423n4/2021-11-nested-findings/issues/175) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache and read from the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

https://github.com/code-423n4/2021-11-nested/blob/f646002b692ca5fa3631acfff87dda897541cf41/contracts/FeeSplitter.sol#L152-L162

```solidity=152
function sendFeesWithRoyalties(
    address _royaltiesTarget,
    IERC20 _token,
    uint256 _amount
) external nonReentrant {
    require(_royaltiesTarget != address(0), "FeeSplitter: INVALID_ROYALTIES_TARGET_ADDRESS");

    _sendFees(_token, _amount, totalWeights);
    _addShares(_royaltiesTarget, _computeShareCount(_amount, royaltiesWeight, totalWeights), address(_token));
}
```

### Recommendation

Change to:

```solidity=152
function sendFeesWithRoyalties(
    address _royaltiesTarget,
    IERC20 _token,
    uint256 _amount
) external nonReentrant {
    require(_royaltiesTarget != address(0), "FeeSplitter: INVALID_ROYALTIES_TARGET_ADDRESS");

    uint256 _totalWeights = totalWeights;

    _sendFees(_token, _amount, _totalWeights);
    _addShares(_royaltiesTarget, _computeShareCount(_amount, royaltiesWeight, _totalWeights), address(_token));
}
```

