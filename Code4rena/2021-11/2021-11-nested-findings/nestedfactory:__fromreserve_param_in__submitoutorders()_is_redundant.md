## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [NestedFactory: _fromReserve param in _submitOutOrders() is redundant](https://github.com/code-423n4/2021-11-nested-findings/issues/128) 

# Handle

GreyArt


# Vulnerability details

## Impact

`_submitOutOrders()` is invoked by 2 functions `sellTokensToNft()` and `sellTokensToWallet()`, both of which specify the `_fromReserve` parameter to be `true`. This parameter is therefore unneeded.

## Recommended Mitigation Steps

```jsx
function _submitOutOrders(
  uint256 _nftId,
  IERC20 _outputToken,
  uint256[] memory _inputTokenAmounts,
  Order[] calldata _orders,
  bool _reserved
) private returns (uint256 feesAmount, uint256 amountBought) {
	...
	
	IERC20 _inputToken = _transferInputTokens(
    _nftId,
    IERC20(_orders[i].token),
    _inputTokenAmounts[i],
    true
	);
}
```

