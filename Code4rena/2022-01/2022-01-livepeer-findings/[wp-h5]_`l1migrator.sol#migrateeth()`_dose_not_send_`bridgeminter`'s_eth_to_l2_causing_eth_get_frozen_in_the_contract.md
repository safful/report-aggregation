## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H5] `L1Migrator.sol#migrateETH()` dose not send `bridgeMinter`'s ETH to L2 causing ETH get frozen in the contract](https://github.com/code-423n4/2022-01-livepeer-findings/issues/205) 

# Handle

WatchPug


# Vulnerability details

Per the `arb-bridge-eth` code:

> all msg.value will deposited to callValueRefundAddress on L2

https://github.com/OffchainLabs/arbitrum/blob/78118ba205854374ed280a27415cb62c37847f72/packages/arb-bridge-eth/contracts/bridge/Inbox.sol#L313

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1ArbitrumMessenger.sol#L65-L74

```solidity
uint256 seqNum = inbox.createRetryableTicket{value: _l1CallValue}(
    target,
    _l2CallValue,
    maxSubmissionCost,
    from,
    from,
    maxGas,
    gasPriceBid,
    data
);
```

At L308-L309, ETH held by `BridgeMinter` is withdrawn to L1Migrator:

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L308-L309

```solidity
        uint256 amount = IBridgeMinter(bridgeMinterAddr)
            .withdrawETHToL1Migrator();
```

However, when calling `sendTxToL2()` the parameter `_l1CallValue` is only the `msg.value`, therefore, the ETH transferred to L2 does not include any funds from `bridgeMinter`. 

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1Migrator.sol#L318-L327

```solidity
    sendTxToL2(
        l2MigratorAddr,
        address(this), // L2 alias of this contract will receive refunds
        msg.value,
        amount,
        _maxSubmissionCost,
        _maxGas,
        _gasPriceBid,
        ""
    )
```

As a result, due to lack of funds, `call` with value = amount to `l2MigratorAddr` will always fail on L2.

Since there is no other way to send ETH to L2, all the ETH from `bridgeMinter` is now frozen in the contract.

### Recommendation

Change to:

```solidity
    sendTxToL2(
        l2MigratorAddr,
        address(this), // L2 alias of this contract will receive refunds
        msg.value + amount, // the `amount` withdrawn from BridgeMinter should be added
        amount,
        _maxSubmissionCost,
        _maxGas,
        _gasPriceBid,
        ""
    )
```

