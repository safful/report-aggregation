## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G21] Cache external call results can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/264) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, external call results should be cached if they are being used for more than one time.

For example:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L153-L158

```solidity
require(
    attributions[msg.sender] > 0 &&
        underlyingValue(msg.sender) >= _amount,
    "ERROR_WITHDRAW-VALUE_BADCONDITOONS"
);
_attributions = (totalAttributions * _amount) / valueAll();
```

In `Vault#withdrawValue()`, `controller.valueAll()` is called twice:

1. L155 `underlyingValue(msg.sender)` -> `valueAll()` -> `controller.valueAll()`;
1. L158 `valueAll()` ->  `controller.valueAll()`.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L400-L411

```solidity
function underlyingValue(address _target)
    public
    view
    override
    returns (uint256)
{
    if (attributions[_target] > 0) {
        return (valueAll() * attributions[_target]) / totalAttributions;
    } else {
        return 0;
    }
}
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L417-L423

```solidity
function valueAll() public view returns (uint256) {
    if (address(controller) != address(0)) {
        return balance + controller.valueAll();
    } else {
        return balance;
    }
}
```

