## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Implementations should inherit their interface](https://github.com/code-423n4/2021-11-malt-findings/issues/242) 

# Handle

WatchPug


# Vulnerability details

It's a best practice for the contract implementations to inherit their interface definition.

Doing so would improve the contract's clarity, and force the implementation to comply with the defined interface.

Instances include:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/TransferService.sol#L14-L14
```solidity=14
contract TransferService is Initializable, Permissions {
```

`TransferService` should inherit `ITransferService`.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/DexHandlers/UniswapHandler.sol#L20-L20

```solidity=20
contract UniswapHandler is Initializable, Permissions {
```

`UniswapHandler` should inherit `IDexHandler`.

