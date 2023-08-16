## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [[WP-M4] Unable to use `L2GatewayRouter` to withdraw LPT from L2 to L1, as `L2LPTGateway` does not implement `L2GatewayRouter` expected method](https://github.com/code-423n4/2022-01-livepeer-findings/issues/202) 

# Handle

WatchPug


# Vulnerability details

Per the document: https://github.com/code-423n4/2022-01-livepeer#l2---l1-lpt-withdrawal

> The following occurs when LPT is withdrawn from L2 to L1:

> The user initiates a withdrawal for X LPT. This can be done in two ways: a. Call outboundTransfer() on L2GatewayRouter which will call outboundTransfer() on L2LPTGateway b. Call outboundTransfer() directly on L2LPTGateway

The method (a) described above won't work in the current implementation due to the missing interface on `L2LPTGateway`.

When initiate a withdraw from the Arbitrum Gateway Router, `L2GatewayRouter` will call `outboundTransfer(address,address,uint256,uint256,uint256,bytes)` on `ITokenGateway(gateway)`:

```solidity
function outboundTransfer(
    address _token,
    address _to,
    uint256 _amount,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    bytes calldata _data
) external payable returns (bytes memory);
```

https://github.com/OffchainLabs/arbitrum/blob/b8366005a697000dda1f57a78a7bdb2313db8fe2/packages/arb-bridge-peripherals/contracts/tokenbridge/arbitrum/gateway/L2GatewayRouter.sol#L57-L64

```solidity
function outboundTransfer(
    address _l1Token,
    address _to,
    uint256 _amount,
    bytes calldata _data
) public payable returns (bytes memory) {
    return outboundTransfer(_l1Token, _to, _amount, 0, 0, _data);
}
```

https://github.com/OffchainLabs/arbitrum/blob/b8366005a697000dda1f57a78a7bdb2313db8fe2/packages/arb-bridge-peripherals/contracts/tokenbridge/libraries/gateway/GatewayRouter.sol#L78-L102

```solidity
function outboundTransfer(
    address _token,
    address _to,
    uint256 _amount,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    bytes calldata _data
) public payable virtual override returns (bytes memory) {
    address gateway = getGateway(_token);
    bytes memory gatewayData = GatewayMessageHandler.encodeFromRouterToGateway(
        msg.sender,
        _data
    );

    emit TransferRouted(_token, msg.sender, _to, gateway);
    return
        ITokenGateway(gateway).outboundTransfer{ value: msg.value }(
            _token,
            _to,
            _amount,
            _maxGas,
            _gasPriceBid,
            gatewayData
        );
}
```

However, `L2LPTGateway` dose not implement `outboundTransfer(address,address,uint256,uint256,uint256,bytes)` but only `outboundTransfer(address,address,uint256,bytes)`:

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/gateway/L2LPTGateway.sol#L65-L89

```solidity
function outboundTransfer(
    address _l1Token,
    address _to,
    uint256 _amount,
    bytes calldata _data
) public override whenNotPaused returns (bytes memory res) {
    // ...
}
```

Therefore, the desired feature to withdraw LPT from L2 to L1 via Arbitrum Router will not be working properly.

## Recommendation

Consider implementing the method used by  Arbitrum Router.

See also the implementation of L2DaiGateway by arbitrum-dai-bridge: https://github.com/makerdao/arbitrum-dai-bridge/blob/master/contracts/l2/L2DaiGateway.sol#L88-L95

