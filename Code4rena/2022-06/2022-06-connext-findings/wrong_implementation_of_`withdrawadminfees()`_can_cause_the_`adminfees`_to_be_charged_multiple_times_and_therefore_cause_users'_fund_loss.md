## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Wrong implementation of `withdrawAdminFees()` can cause the `adminFees` to be charged multiple times and therefore cause users' fund loss](https://github.com/code-423n4/2022-06-connext-findings/issues/202) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/SwapUtils.sol#L1053-L1062


# Vulnerability details

```solidity
function withdrawAdminFees(Swap storage self, address to) internal {
  IERC20[] memory pooledTokens = self.pooledTokens;
  for (uint256 i = 0; i < pooledTokens.length; i++) {
    IERC20 token = pooledTokens[i];
    uint256 balance = self.adminFees[i];
    if (balance != 0) {
      token.safeTransfer(to, balance);
    }
  }
}
```

`self.adminFees[i]` should be reset to 0 every time it's withdrawn. Otherwise, the `adminFees` can be withdrawn multiple times.

The admin may just be unaware of this issue and casualty `withdrawAdminFees()` from time to time, and rug all the users slowly.

### Recommendation

Change to:

```solidity
function withdrawAdminFees(Swap storage self, address to) internal {
  IERC20[] memory pooledTokens = self.pooledTokens;
  for (uint256 i = 0; i < pooledTokens.length; i++) {
    IERC20 token = pooledTokens[i];
    uint256 balance = self.adminFees[i];
    if (balance != 0) {
      self.adminFees[i] = 0;
      token.safeTransfer(to, balance);
    }
  }
}
```

