## Tags

- bug
- disagree with severity
- sponsor confirmed
- 0 (Non-critical)

# [Missing events for critical operations](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/55) 

# Handle

WatchPug


# Vulnerability details

Across the contracts, there are certain critical operations that change critical values that affect the users of the protocol.

It's a best practice for these setter functions to emit events to record these changes on-chain for off-chain monitors/tools/interfaces to register the updates and react if necessary.

Instances include:

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L322-L324
```solidity=322
function setGovernance(address _governance) external onlyGovernance {
    _setGovernance(_governance);
}
```

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L331-L333
```solidity=331
function approveContractAccess(address account) external onlyGovernance {
    _approveContractAccess(account);
}
```

https://github.com/Badger-Finance/ibbtc/blob/d8b95e8d145eb196ba20033267a9ba43a17be02c/contracts/Zap.sol#L335-L337
```solidity=335
function revokeContractAccess(address account) external onlyGovernance {
    _revokeContractAccess(account);
}
```


https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/6f700995129182fec81b772f97abab9977b46026/contracts/IbbtcVaultZap.sol#L116-L119
```solidity=116
function setGuardian(address _guardian) external {
    _onlyGovernance();
    governance = _guardian;
}
```


https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/6f700995129182fec81b772f97abab9977b46026/contracts/IbbtcVaultZap.sol#L121-L124
```solidity=121
function setGovernance(address _governance) external {
    _onlyGovernance();
    governance = _governance;
}
```



https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/a5c71b72222d84b6414ca0339ed1761dc79fe56e/contracts/SettToRenIbbtcZap.sol#L130-L133
```solidity=130
function setGuardian(address _guardian) external {
    _onlyGovernance();
    governance = _guardian;
}
```

https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/a5c71b72222d84b6414ca0339ed1761dc79fe56e/contracts/SettToRenIbbtcZap.sol#L135-L138
```solidity=135
function setGovernance(address _governance) external {
    _onlyGovernance();
    governance = _governance;
}
```


