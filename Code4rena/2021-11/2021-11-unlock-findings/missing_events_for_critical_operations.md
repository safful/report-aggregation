## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing events for critical operations](https://github.com/code-423n4/2021-11-unlock-findings/issues/204) 

# Handle

WatchPug


# Vulnerability details

Across the contracts, there are certain critical operations that change critical values that affect the users of the protocol.

It's a best practice for these setter functions to emit events to record these changes on-chain for off-chain monitors/tools/interfaces to register the updates and react if necessary.

Instances include:

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockMetadata.sol#L54-L60

```solidity=54
  function updateLockName(
    string calldata _lockName
  ) external
    onlyLockManager
  {
    name = _lockName;
  }
```

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockMetadata.sol#L92-L98

```solidity=92
  function setBaseTokenURI(
    string calldata _baseTokenURI
  ) external
    onlyLockManager
  {
    baseTokenURI = _baseTokenURI;
  }
```

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinLockCore.sol#L204-L214

```solidity=204
  function setEventHooks(
    address _onKeyPurchaseHook,
    address _onKeyCancelHook
  ) external
    onlyLockManager()
  {
    require(_onKeyPurchaseHook == address(0) || _onKeyPurchaseHook.isContract(), 'INVALID_ON_KEY_SOLD_HOOK');
    require(_onKeyCancelHook == address(0) || _onKeyCancelHook.isContract(), 'INVALID_ON_KEY_CANCEL_HOOK');
    onKeyPurchaseHook = ILockKeyPurchaseHook(_onKeyPurchaseHook);
    onKeyCancelHook = ILockKeyCancelHook(_onKeyCancelHook);
  }
```

