## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong implementation of `NoYield.sol#emergencyWithdraw()`](https://github.com/code-423n4/2021-12-sublime-findings/issues/115) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/NoYield.sol#L78-L83

```solidity=78{81}
function emergencyWithdraw(address _asset, address payable _wallet) external onlyOwner returns (uint256 received) {
    require(_wallet != address(0), 'cant burn');
    uint256 amount = IERC20(_asset).balanceOf(address(this));
    IERC20(_asset).safeTransfer(_wallet, received);
    received = amount;
}
```

`received` is not being assigned prior to L81, therefore, at L81, `received` is `0`.

As a result, the `emergencyWithdraw()` does not work, in essence.

### Recommendation

Change to:

```solidity=78
function emergencyWithdraw(address _asset, address payable _wallet) external onlyOwner returns (uint256 received) {
    require(_wallet != address(0), 'cant burn');
    received = IERC20(_asset).balanceOf(address(this));
    IERC20(_asset).safeTransfer(_wallet, received);
}
```

