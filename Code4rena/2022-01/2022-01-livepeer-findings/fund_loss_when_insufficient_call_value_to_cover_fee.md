## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Fund loss when insufficient call value to cover fee](https://github.com/code-423n4/2022-01-livepeer-findings/issues/238) 

# Handle

gzeon


# Vulnerability details

## Impact
Fund can be lost if the L1 call value provided is insufficient to cover `_maxSubmissionCost`, or stuck if insufficient to cover `_maxSubmissionCost + (_maxGas * _gasPriceBid)`.

## Proof of Concept
`outboundTransfer` in `L1LPTGateway` does not check if the call value is sufficient, if it is `< _maxSubmissionCost` the retryable ticket creation will fail and fund is lost; if it is `<_maxSubmissionCost + (_maxGas * _gasPriceBid)` the ticket would require manual execution.

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L1/gateway/L1LPTGateway.sol#L80
```
    function outboundTransfer(
        address _l1Token,
        address _to,
        uint256 _amount,
        uint256 _maxGas,
        uint256 _gasPriceBid,
        bytes calldata _data
    ) external payable override whenNotPaused returns (bytes memory) {
        require(_l1Token == l1Lpt, "TOKEN_NOT_LPT");

        // nested scope to avoid stack too deep errors
        address from;
        uint256 seqNum;
        bytes memory extraData;
        {
            uint256 maxSubmissionCost;
            (from, maxSubmissionCost, extraData) = parseOutboundData(_data);
            require(extraData.length == 0, "CALL_HOOK_DATA_NOT_ALLOWED");

            // transfer tokens to escrow
            TokenLike(_l1Token).transferFrom(from, l1LPTEscrow, _amount);

            bytes memory outboundCalldata = getOutboundCalldata(
                _l1Token,
                from,
                _to,
                _amount,
                extraData
            );

            seqNum = sendTxToL2(
                l2Counterpart,
                from,
                maxSubmissionCost,
                _maxGas,
                _gasPriceBid,
                outboundCalldata
            );
        }

        emit DepositInitiated(_l1Token, from, _to, seqNum, _amount);

        return abi.encode(seqNum);
    }
```

## Recommended Mitigation Steps
Add check similar to the one used in `L1GatewayRouter` provided by Arbitrum team

https://github.com/OffchainLabs/arbitrum/blob/b8366005a697000dda1f57a78a7bdb2313db8fe2/packages/arb-bridge-peripherals/contracts/tokenbridge/ethereum/gateway/L1GatewayRouter.sol#L236
```
        uint256 expectedEth = _maxSubmissionCost + (_maxGas * _gasPriceBid);
        require(_maxSubmissionCost > 0, "NO_SUBMISSION_COST");
        require(msg.value == expectedEth, "WRONG_ETH_VALUE");
```

