## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [[WP-N5] Missing error messages in require statements](https://github.com/code-423n4/2022-01-insure-findings/issues/217) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L288-L288

```solidity
require(registry.isListed(msg.sender));
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Factory.sol#L100-L100

```solidity
require(address(_template) != address(0));
```

