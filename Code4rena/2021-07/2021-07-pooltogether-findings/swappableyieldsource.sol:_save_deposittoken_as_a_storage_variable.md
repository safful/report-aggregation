## Tags

- bug
- G (Gas Optimization)
- SwappableYieldSource
- sponsor confirmed

# [SwappableYieldSource.sol: Save depositToken as a storage variable](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/26) 

# Handle

hickuphh3


# Vulnerability details

### Impact

Assuming that `depositToken` of a yield source doesn't change, it would make sense to save its value as a storage variable in the contract as well, so that an external call to `yieldSource` to retrieve it can be avoided whenever it is needed.

### Recommended Mitigation Steps

Define `address public override depositToken;` or `IERC20Upgradeable public depositToken;` which gets initialized in the `initialize()` function. The nice thing is that it also doesn't need to be updated when swapping sources because a requirement is that the new yield source must have the same deposit token.

As an optimization, since the `_requireYieldSource()` function already retrieves the `depositToken` address, it can return it so that its value need not be externally retrieved again in the `initialize()` function.

The `depositToken()` function can be removed if the former suggestion is implemented (ie. `address public override depositToken`).

Then, `yieldSource.depositToken()` can be replaced with `depositToken` where applicable (with appropriate casting).

A part of the former implementation is provided below.

```jsx
address public override depositToken;

function initialize(...) {
	address depositTokenAddress = _requireYieldSource(_yieldSource);
	yieldSource = _yieldSource;
  depositToken = depositTokenAddress;
	...
	IERC20Upgradeable(depositTokenAddress).safeApprove(address(_yieldSource), type(uint256).max);
}

function _requireYieldSource(IYieldSource _yieldSource) internal view returns (address depositTokenAddress) {
	...
	(depositTokenAddress) = abi.decode(depositTokenAddressData, (address));
}

// function depositToken() can be removed
// yieldSource.depositToken() can be replaced with depositToken in other functions
// Example: _setYieldSource
function _setYieldSource(IYieldSource _newYieldSource) internal {
	_requireDifferentYieldSource(_newYieldSource);
	// Commented out check below should be shifted to inside _requireDifferentYieldSource()
	// Optimization: it can also return depositToken to avoid another SLOAD
	// similar to _requireYieldSource() above
	// require(_newYieldSource.depositToken() == depositToken, "SwappableYieldSource/different-deposit-token");

  yieldSource = _newYieldSource;
  IERC20Upgradeable(depositToken).safeApprove(address(_newYieldSource), type(uint256).max);

  emit SwappableYieldSourceSet(_newYieldSource);
}
```

