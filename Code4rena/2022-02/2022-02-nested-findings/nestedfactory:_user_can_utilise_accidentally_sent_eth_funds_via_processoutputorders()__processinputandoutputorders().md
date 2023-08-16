## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [NestedFactory: User can utilise accidentally sent ETH funds via processOutputOrders() / processInputAndOutputOrders()](https://github.com/code-423n4/2022-02-nested-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L71
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L286-L296
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L370-L375
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L482-L492


# Vulnerability details

## Impact

Should a user accidentally send ETH to the `NestedFactory`, anyone can utilise it to their own benefit by calling `processOutputOrders()` / `processInputAndOutputOrders()`. This is possible because:

1. `receive()` has no restriction on the sender
2. `processOutputOrders()` does not check `msg.value`, and rightly so, because funds are expected to come from `reserve`.
3. `transferInputTokens()` does not handle the case where `ETH` could be specified as an address by the user for an output order.

```jsx
if (address(_inputToken) == ETH) {
  require(address(this).balance >= _inputTokenAmount, "NF: INVALID_AMOUNT_IN");
  weth.deposit{ value: _inputTokenAmount }();
  return (IERC20(address(weth)), _inputTokenAmount);
}
```

Hence, the attack vector is simple. Should a user accidentally send ETH to the contract, create an output `Order` with `token` being `ETH` and amount corresponding to the NestedFactory’s ETH balance.

## Recommended Mitigation Steps

1. Since plain / direct`ETH` transfers are only expected to solely come from `weth` (excluding payable functions), we recommend restricting the sender to be `weth`, like how it is done in `[FeeSplitter](https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L101-L104)`.
    
    We are aware that this was raised previously here: https://github.com/code-423n4/2021-11-nested-findings/issues/188 and would like to add that the restricting the sender in the `receive()` function will not affect `payable` functions. From from what we see, plain ETH transfers are also not expected to come from other sources like `NestedReserve` or operators.
    

```jsx
receive() external payable {
  require(msg.sender == address(weth), "NF: ETH_SENDER_NOT_WETH");
}
```

1. Check that `_fromReserve` is false in the scenario `address(_inputToken) == ETH`.

```jsx
if (address(_inputToken) == ETH) {
  require(!_fromReserve, "NF: INVALID_INPUT_TOKEN");
  require(address(this).balance >= _inputTokenAmount, "NF: INVALID_AMOUNT_IN");
  weth.deposit{ value: _inputTokenAmount }();
  return (IERC20(address(weth)), _inputTokenAmount);
}
```

