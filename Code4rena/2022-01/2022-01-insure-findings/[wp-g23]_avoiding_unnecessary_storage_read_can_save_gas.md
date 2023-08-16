## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G23] Avoiding unnecessary storage read can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/265) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Ownership.sol#L17-L20

```solidity
constructor() {
    _owner = msg.sender;
    emit AcceptNewOwnership(_owner);
}
```

At L19, the parameter of `AcceptNewOwnership` can use `msg.sender` directly to avoid unnecessary storage read of `_owner` to save some gas.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Ownership.sol#L65-L71

```solidity
function acceptTransferOwnership() external override onlyFutureOwner {
    /***
        *@notice Accept a transfer of ownership
        */
    _owner = _futureOwner;
    emit AcceptNewOwnership(_owner);
}
```

At L69, `_futureOwner` can use `msg.sender` directly to avoid unnecessary storage read of `_futureOwner` to save some gas.

As `onlyFutureOwner()` ensures that `require(_futureOwner == msg.sender, "...");`.

