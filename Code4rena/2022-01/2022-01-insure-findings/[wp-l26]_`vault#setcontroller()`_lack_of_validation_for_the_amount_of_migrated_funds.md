## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [[WP-L26] `Vault#setController()` Lack of validation for the amount of migrated funds](https://github.com/code-423n4/2022-01-insure-findings/issues/268) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L485-L496

```solidity
function setController(address _controller) public override onlyOwner {
    require(_controller != address(0), "ERROR_ZERO_ADDRESS");

    if (address(controller) != address(0)) {
        controller.migrate(address(_controller));
        controller = IController(_controller);
    } else {
        controller = IController(_controller);
    }

    emit ControllerSet(_controller);
}
```

`controller.migrate()` is a critical operation, we recommend adding validation for the amount of migrated funds.

### Recommendation

Can be changed to:

```solidity
function setController(address _controller) public override onlyOwner {
    require(_controller != address(0), "ERROR_ZERO_ADDRESS");

    if (address(controller) != address(0)) {
        uint256 beforeUnderlying = controller.valueAll();
        controller.migrate(address(_controller));
        require(IController(_controller).valueAll() >= beforeUnderlying, "...");
        controller = IController(_controller);
    } else {
        controller = IController(_controller);
    }

    emit ControllerSet(_controller);
}
```

