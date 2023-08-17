## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-02

# [When a smart contract calls CrossChainRelayerArbitrum.processCalls, excess submission fees may be lost](https://github.com/code-423n4/2022-12-pooltogether-findings/issues/63) 

# Lines of code

https://github.com/pooltogether/ERC5164/blob/5647bd84f2a6d1a37f41394874d567e45a97bf48/src/ethereum-arbitrum/EthereumToArbitrumRelayer.sol#L118-L127


# Vulnerability details

## Impact
When the user calls CrossChainRelayerArbitrum.processCalls, ETH is sent as the submission fee. 
According to the documentation : https://github.com/OffchainLabs/arbitrum/blob/master/docs/L1_L2_Messages.md#retryable-transaction-lifecycle
```
Credit-Back Address: Address to which all excess gas is credited on L2; i.e., excess ETH for base submission cost (MaxSubmissionCost - ActualSubmissionCostPaid) and excess ETH provided for L2 execution ( (GasPrice x MaxGas) - ActualETHSpentInExecution).
...
Submission fee is collected: submission fee is deducted from the sender’s L2 account; MaxSubmissionCost - submission fee is credited to Credit-Back Address.
```
the excess submission fee is refunded to the address on L2 of the excessFeeRefundAddress provided when calling createRetryableTicket. 
```solidity
     * @notice Put a message in the L2 inbox that can be reexecuted for some fixed amount of time if it reverts
     * @dev all msg.value will deposited to callValueRefundAddress on L2
     * @param destAddr destination L2 contract address
     * @param l2CallValue call value for retryable L2 message
     * @param  maxSubmissionCost Max gas deducted from user's L2 balance to cover base submission fee
     * @param excessFeeRefundAddress maxgas x gasprice - execution cost gets credited here on L2 balance
     * @param callValueRefundAddress l2Callvalue gets credited here on L2 if retryable txn times out or gets cancelled
     * @param maxGas Max gas deducted from user's L2 balance to cover L2 execution
     * @param gasPriceBid price bid for L2 execution
     * @param data ABI encoded data of L2 message
     * @return unique id for retryable transaction (keccak256(requestID, uint(0) )
     */
    function createRetryableTicket(
        address destAddr,
        uint256 l2CallValue,
        uint256 maxSubmissionCost,
        address excessFeeRefundAddress,
        address callValueRefundAddress,
        uint256 maxGas,
        uint256 gasPriceBid,
        bytes calldata data
    ) external payable virtual override onlyWhitelisted returns (uint256) {
```
In CrossChainRelayerArbitrum.processCalls, excessFeeRefundAddress == msg.sender.
```solidity
    uint256 _ticketID = inbox.createRetryableTicket{ value: msg.value }(
      address(executor),
      0,
      _maxSubmissionCost,
      msg.sender,   // @audit : excessFeeRefundAddress
      msg.sender,  // @audit: callValueRefundAddress
      _gasLimit,
      _gasPriceBid,
      _data
    );
```
For EOA accounts, the excess submission fees are correctly refunded to their address on L2.
However, for smart contracts, since there may not exist a corresponding address on L2, these excess submission fees will be lost.

Also, since the callValueRefundAddress is also msg.sender, according to the documentation, if the Retryable Ticket is cancelled or expired, then the smart contract caller may lose all the submission fees
```
If the Retryable Ticket is cancelled or expires before it is redeemed, Callvalue is credited to Beneficiary.

```

## Proof of Concept
https://github.com/pooltogether/ERC5164/blob/5647bd84f2a6d1a37f41394874d567e45a97bf48/src/ethereum-arbitrum/EthereumToArbitrumRelayer.sol#L118-L127
https://github.com/OffchainLabs/arbitrum/blob/master/packages/arb-bridge-eth/contracts/bridge/Inbox.sol#L333-L354
## Tools Used
None
## Recommended Mitigation Steps
Consider allowing the user to specify excessFeeRefundAddress and callValueRefundAddress when calling CrossChainRelayerArbitrum.processCalls 