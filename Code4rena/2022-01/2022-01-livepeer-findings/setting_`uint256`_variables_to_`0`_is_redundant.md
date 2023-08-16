## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Setting `uint256` variables to `0` is redundant](https://github.com/code-423n4/2022-01-livepeer-findings/issues/124) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L471-L471

```solidity
uint256 total = 0;
```

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L472-L472

```solidity
for (uint256 i = 0; i < _unbondingLockIds.length; i++)
```

Setting `uint256` variables to `0` is redundant as they default to `0`.

