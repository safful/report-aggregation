## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Overpayment of native ETH is not refunded to buyer](https://github.com/code-423n4/2022-06-infinity-findings/issues/244) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L119-L121
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1228-L1232


# Vulnerability details

`InfinityExchange` accepts payments in native ETH, but does not return overpayments to the buyer. Overpayments are likely in the case of auction orders priced in native ETH.

In the case of a Dutch or reverse Dutch auction priced in native ETH, the end user is likely to send more ETH than the final calculated price in order to ensure their transaction succeeds, since price is a function of `block.timestamp`, and the user cannot predict the timestamp at which their transaction will be included. 

In a Dutch auction, final price may decrease below the calculated price at the time the transaction is sent. In a reverse Dutch auction, the price may increase above the calculated price by the time a transaction is included, so the buyer is incentivized to provide additional ETH in case the price rises while their transaction is waiting for inclusion.

The `takeOrders` and `takeMultipleOneOrders` functions both check that the buyer has provided an ETH amount greater than or equal to the total price at the time of execution:

[`InfinityExchange#takeOrders`](https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L359-L363)

```solidity
    // check to ensure that for ETH orders, enough ETH is sent
    // for non ETH orders, IERC20 safeTransferFrom will throw error if insufficient amount is sent
    if (isMakerSeller && currency == address(0)) {
      require(msg.value >= totalPrice, 'invalid total price');
    }
```

[`InfinityExchange#takeMultipleOneOrders`](https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L323-L327)

```solidity
    // check to ensure that for ETH orders, enough ETH is sent
    // for non ETH orders, IERC20 safeTransferFrom will throw error if insufficient amount is sent
    if (isMakerSeller && currency == address(0)) {
      require(msg.value >= totalPrice, 'invalid total price');
    }
```

However, neither of these functions refunds the user in the case of overpayment. Instead, overpayment amounts will accrue in the contract balance.

Moreover, since there is a bug in `rescueETH` that prevents ether withdrawals from `InfinityExchange`, these overpayments will be locked permanently: the owner cannot withdraw and refund overpayments manually.

Scenario:
- Alice creates a sell order for her token with constraints that set up a reverse Dutch auction: start price `500`, end price `2000`, start time `1`, end time `5`.
- Bob fills the order at time `2`. The calculated price is `875`. Bob is unsure when his transaction will be included, so provides a full `2000` wei payment.
- Bob's transaction is included at time `3`. The calculated price is `1250`.
- Bob's additional `750` wei are locked in the contract and not refunded.

Suggestion: Calculate and refund overpayment amounts to callers.

