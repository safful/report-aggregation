## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`AaveVault.sol#_pull()` may return wrong `actualTokenAmounts`](https://github.com/code-423n4/2021-12-mellow-findings/issues/119) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/test_brownie/contracts/AaveVault.sol#L80-L94

```solidity=80
function _pull(
        address to,
        uint256[] memory tokenAmounts,
        bytes memory
    ) internal override returns (uint256[] memory actualTokenAmounts) {
        address[] memory tokens = _vaultTokens;
        for (uint256 i = 0; i < _aTokens.length; i++) {
            if ((_tvls[i] == 0) || (tokenAmounts[i] == 0)) {
                continue;
            }
            _lendingPool().withdraw(tokens[i], tokenAmounts[i], to);
        }
        updateTvls();
        actualTokenAmounts = tokenAmounts;
    }
```

In Aave LendingPool, the actual amount withdrawn may be different from the requested amount, we suggest using the return amount as `actualTokenAmount`.

https://github.com/aave/protocol-v2/blob/master/contracts/protocol/lendingpool/LendingPool.sol#L155-L157

```solidity
if (amount == type(uint256).max) {
    amountToWithdraw = userBalance;
}
```

### Recommendation

Change to:

```solidity
function _pull(
        address to,
        uint256[] memory tokenAmounts,
        bytes memory
    ) internal override returns (uint256[] memory actualTokenAmounts) {
        address[] memory tokens = _vaultTokens;
        for (uint256 i = 0; i < _aTokens.length; i++) {
            if ((_tvls[i] == 0) || (tokenAmounts[i] == 0)) {
                continue;
            }
            tokenAmounts[i] = _lendingPool().withdraw(tokens[i], tokenAmounts[i], to);
        }
        updateTvls();
        actualTokenAmounts = tokenAmounts;
    }
```

