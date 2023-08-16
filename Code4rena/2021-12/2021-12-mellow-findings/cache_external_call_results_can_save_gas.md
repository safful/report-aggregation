## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache external call results can save gas](https://github.com/code-423n4/2021-12-mellow-findings/issues/89) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, external call results should be cached if they are being used for more than one time.

For example:

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/AaveVault.sol#L101-L109

```solidity=101
function _allowTokenIfNecessary(address token) internal {
    if (IERC20(token).allowance(address(this), address(_lendingPool())) < type(uint256).max / 2) {
        IERC20(token).approve(address(_lendingPool()), type(uint256).max);
    }
}

function _lendingPool() internal view returns (ILendingPool) {
    return IAaveVaultGovernance(address(_vaultGovernance)).delayedProtocolParams().lendingPool;
}
```

Considering that `_lendingPool()` is a internal call that includes a storage read of `_vaultGovernance` and an external call of `IAaveVaultGovernance.delayedProtocolParams()`. Cache the result of `_lendingPool()` in the stack can save some gas.

### Recommendation

Change to:

```solidity
function _allowTokenIfNecessary(address token) internal {
    address lendingPool = address(_lendingPool());
    if (IERC20(token).allowance(address(this), lendingPool) < type(uint256).max / 2) {
        IERC20(token).approve(lendingPool, type(uint256).max);
    }
}
```

