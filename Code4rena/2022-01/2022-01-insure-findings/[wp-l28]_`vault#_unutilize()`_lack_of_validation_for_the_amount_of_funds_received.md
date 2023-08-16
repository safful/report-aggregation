## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [[WP-L28] `Vault#_unutilize()` Lack of validation for the amount of funds received](https://github.com/code-423n4/2022-01-insure-findings/issues/270) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L429-L434

```solidity
function _unutilize(uint256 _amount) internal {
    require(address(controller) != address(0), "ERROR_CONTROLLER_NOT_SET");

    controller.withdraw(address(this), _amount);
    balance += _amount;
}
```

### Recommendation

Can be changed to:

```solidity
function _unutilize(uint256 _amount) internal {
    require(address(controller) != address(0), "ERROR_CONTROLLER_NOT_SET");

    uint256 beforeBalance = IERC20(token).balanceOf(address(this));
    controller.withdraw(address(this), _amount);
    uint256 received = IERC20(token).balanceOf(address(this)) - beforeBalance;
    require(received >= _amount, "...");
    balance += received;
}
```


