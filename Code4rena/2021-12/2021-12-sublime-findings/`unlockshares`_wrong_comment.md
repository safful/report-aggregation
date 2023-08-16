## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`unlockShares` wrong comment](https://github.com/code-423n4/2021-12-sublime-findings/issues/135) 

# Handle

cmichel


# Vulnerability details

The strategy contracts define an `unlockShares` function that must accept an `asset` parameter as the **share** token (yield token, aToken, cToken, etc.), otherwise, the code does not work.
However, all comments say that `asset` is the address of the **underlying token**.

```solidity
/**
  * @notice Used to unlock shares
  * @param asset the address of underlying token
  * @param amount the amount of shares to unlock
  * @return received amount of shares received
  **/
function unlockShares(address asset, uint256 amount) external override onlySavingsAccount nonReentrant returns (uint256) {
    if (amount == 0) {
        return 0;
    }
}
```

## Recommended Mitigation Steps
Fix the comments for all `unlockShares` by saying `asset` is the share token, not the underlying token.


