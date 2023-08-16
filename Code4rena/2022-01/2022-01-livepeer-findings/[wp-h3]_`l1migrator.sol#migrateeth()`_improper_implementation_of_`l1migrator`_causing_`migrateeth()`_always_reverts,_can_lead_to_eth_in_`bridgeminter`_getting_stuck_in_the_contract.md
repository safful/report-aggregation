## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [[WP-H3] `L1Migrator.sol#migrateETH()` Improper implementation of `L1Migrator` causing `migrateETH()` always reverts, can lead to ETH in `BridgeMinter` getting stuck in the contract](https://github.com/code-423n4/2022-01-livepeer-findings/issues/198) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L308-L310

```solidity
uint256 amount = IBridgeMinter(bridgeMinterAddr)
            .withdrawETHToL1Migrator();
```

`L1Migrator.sol#migrateETH()` will call `IBridgeMinter(bridgeMinterAddr).withdrawETHToL1Migrator()` to withdraw ETH from `BridgeMinter`.

However, the current implementation of `L1Migrator` is unable to receive ETH.

https://github.com/livepeer/protocol/blob/20e7ebb86cdb4fe9285bf5fea02eb603e5d48805/contracts/token/BridgeMinter.sol#L94-L94

```solidity
(bool ok, ) = l1MigratorAddr.call.value(address(this).balance)("");
```

A contract receiving Ether must have at least one of the functions below:

- `receive() external payable`
- `fallback() external payable`

`receive()` is called if `msg.data` is empty, otherwise `fallback()` is called.

Because `L1Migrator` implement neither `receive()` or `fallback()`, the `call` at L94 will always revert.

## Impact

All the ETH held by the `BridgeMinter` can get stuck in the contract.

## Recommandation

Add `receive() external payable {}` in `L1Migrator`.

