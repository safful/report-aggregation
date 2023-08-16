## Tags

- bug
- 3 (High Risk)
- SwappableYieldSource
- sponsor confirmed

# [SwappableYieldSource: Missing same deposit token check in transferFunds()](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/29) 

# Handle

hickuphh3


# Vulnerability details

### Impact

`transferFunds()` will transfer funds from a specified yield source `_yieldSource` to the current yield source set in the contract `_currentYieldSource`. However, it fails to check that the deposit tokens are the same. If the specified yield source's assets are of a higher valuation, then a malicious owner or asset manager will be able to exploit and pocket the difference.

### Proof of Concept

Assumptions:

- `_yieldSource` has a deposit token of WETH (18 decimals)
- `_currentYieldSource` has a deposit token of DAI (18 decimals)
- 1 WETH > 1 DAI (definitely true, I'd be really sad otherwise)

Attacker does the following:

1. Deposit 100 DAI into the swappable yield source contract
2. Call `transferFunds(_yieldSource, 100 * 1e18)`
    - `_requireDifferentYieldSource()` passes
    - `_transferFunds(_yieldSource, 100 * 1e18)` is called
        - `_yieldSource.redeemToken(_amount);` → This will transfer 100 WETH out of the `_yieldSource` into the contract
        - `uint256 currentBalance = IERC20Upgradeable(_yieldSource.depositToken()).balanceOf(address(this));` → This will equate to ≥ 100 WETH.
        - `require(_amount <= currentBalance, "SwappableYieldSource/transfer-amount-different");` is true since both are `100 * 1e18`
        - `_currentYieldSource.supplyTokenTo(currentBalance, address(this));` → This supplies the transferred 100 DAI from step 1 to the current yield source
    - We now have 100 WETH in the swappable yield source contract
3. Call `transferERC20(WETH, attackerAddress, 100 * 1e18)` to withdraw 100 WETH out of the contract to the attacker's desired address. 

### Recommended Mitigation Steps

`_requireDifferentYieldSource()` should also verify that the yield sources' deposit token addresses are the same.

```jsx
function _requireDifferentYieldSource(IYieldSource _yieldSource) internal view {
    require(address(_yieldSource) != address(yieldSource), "SwappableYieldSource/same-yield-source");
		require(_newYieldSource.depositToken() == yieldSource.depositToken(), "SwappableYieldSource/different-deposit-token");
}
```

