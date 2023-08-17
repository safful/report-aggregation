## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Accumulated ETH fees of InfinityExchange cannot be retrieved](https://github.com/code-423n4/2022-06-infinity-findings/issues/296) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1228-L1232


# Vulnerability details

ETH fees accumulated from takeOrders() and takeMultipleOneOrders() operations are permanently frozen within the contract as there is only one way designed to retrieve them, a rescueETH() function, and it will work as intended, not being able to access ETH balance of the contract.

Setting the severity as high as the case is a violation of system's core logic and a permanent freeze of ETH revenue of the project.

## Proof of Concept

Fees are accrued in user-facing takeOrders() and takeMultipleOneOrders() via the following call sequences:

```
takeOrders -> _takeOrders -> _execTakeOrders -> _transferNFTsAndFees -> _transferFees
takeMultipleOneOrders -> _execTakeOneOrder -> _transferNFTsAndFees -> _transferFees
```

While token fees are transferred right away, ETH fees are kept with the InfinityExchange contract:

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1119-L1141

```solidity
  /**
   * @notice Transfer fees. Fees are always transferred from buyer to the seller and the exchange although seller is 
            the one that actually 'pays' the fees
   * @dev if the currency ETH, no additional transfer is needed to pay exchange fees since the contract is 'payable'
   * @param seller the seller
   * @param buyer the buyer
   * @param amount amount to transfer
   * @param currency currency of the transfer
   */
  function _transferFees(
    address seller,
    address buyer,
    uint256 amount,
    address currency
  ) internal {
    // protocol fee
    uint256 protocolFee = (PROTOCOL_FEE_BPS * amount) / 10000;
    uint256 remainingAmount = amount - protocolFee;
    // ETH
    if (currency == address(0)) {
      // transfer amount to seller
      (bool sent, ) = seller.call{value: remainingAmount}('');
      require(sent, 'failed to send ether to seller');
```

I.e. when `currency` is ETH the fee part of the amount, `protocolFee`, is left with the InfinityExchange contract.

The only way to retrieve ETH from the contract is rescueETH() function:

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1228-L1232

```solidity
  /// @dev used for rescuing exchange fees paid to the contract in ETH
  function rescueETH(address destination) external payable onlyOwner {
    (bool sent, ) = destination.call{value: msg.value}('');
    require(sent, 'failed');
  }
```

However, it cannot reach ETH on the contract balance as `msg.value` is used as the amount to be sent over. I.e. only ETH attached to the rescueETH() call is transferred from `owner` to `destination`. ETH funds that InfinityExchange contract holds remain inaccessible.

## Recommended Mitigation Steps

Consider adding contract balance to the funds transferred:

```solidity
  /// @dev used for rescuing exchange fees paid to the contract in ETH
  function rescueETH(address destination) external payable onlyOwner {
-   (bool sent, ) = destination.call{value: msg.value}('');
+   (bool sent, ) = destination.call{value: address(this).balance}('');
    require(sent, 'failed');
  }
```

