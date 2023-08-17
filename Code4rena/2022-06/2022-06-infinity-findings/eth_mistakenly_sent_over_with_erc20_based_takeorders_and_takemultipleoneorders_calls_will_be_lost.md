## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ETH mistakenly sent over with ERC20 based takeOrders and takeMultipleOneOrders calls will be lost](https://github.com/code-423n4/2022-06-infinity-findings/issues/346) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L323-L327
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L359-L363


# Vulnerability details

takeOrders() and takeMultipleOneOrders() are the main user facing functionality of the protocol. Both require `currency` to be fixed for the call and can have it either as a ERC20 token or ETH. This way, the probability of a user sending over a ETH with the call whose `currency` is a ERC20 token isn't negligible. However, in this case ETH funds of a user will be permanently lost.

Setting the severity to medium as this is permanent fund freeze scenario conditional on a user mistake, which probability can be deemed high enough as the same functions are used for ETH and ERC20 orders.

## Proof of Concept

Both takeOrders() and takeMultipleOneOrders() only check that ETH funds are enough to cover the order's `totalPrice`:

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L323-L327

```solidity
    // check to ensure that for ETH orders, enough ETH is sent
    // for non ETH orders, IERC20 safeTransferFrom will throw error if insufficient amount is sent
    if (isMakerSeller && currency == address(0)) {
      require(msg.value >= totalPrice, 'invalid total price');
    }
```

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L359-L363

```solidity
    // check to ensure that for ETH orders, enough ETH is sent
    // for non ETH orders, IERC20 safeTransferFrom will throw error if insufficient amount is sent
    if (isMakerSeller && currency == address(0)) {
      require(msg.value >= totalPrice, 'invalid total price');
    }
```

When `currency` is some ERC20 token, while `msg.value > 0`, the `msg.value` will be permanently frozen within the contract.

## Recommended Mitigation Steps

Consider adding the check for `msg.value` to be zero for the cases when it is not utilized:

```solidity
    // check to ensure that for ETH orders, enough ETH is sent
    // for non ETH orders, IERC20 safeTransferFrom will throw error if insufficient amount is sent
    if (isMakerSeller && currency == address(0)) {
      require(msg.value >= totalPrice, 'invalid total price');
    } else {
      require(msg.value == 0, 'non-zero ETH value');
    }
```

