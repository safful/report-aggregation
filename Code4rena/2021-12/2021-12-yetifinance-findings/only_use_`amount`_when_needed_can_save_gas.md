## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Only use `amount` when needed can save gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/279) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/TroveManagerLiquidations.sol#L839-L847

```solidity=839
function _updateWAssetsRewardOwner(newColls memory _colls, address _borrower, address _newOwner) internal {
        for (uint i = 0; i < _colls.tokens.length; i++) {
            address token = _colls.tokens[i];
            uint amount = _colls.amounts[i];
            if (whitelist.isWrapped(token)) {
                IWAsset(token).updateReward(_borrower, _newOwner, amount);
            }
        }
    }
```

Since `amount` is only needed in `if (whitelist.isWrapped(token)) {...}`, so `uint amount = _colls.amounts[i];` should be moved to inside `if (whitelist.isWrapped(token)) {...}`.

Furthermore, considering that `amount` is only used once, it can be replaced with `_colls.amounts[i]`.

### Recommendation

Change to:

```solidity=839
function _updateWAssetsRewardOwner(newColls memory _colls, address _borrower, address _newOwner) internal {
    for (uint i = 0; i < _colls.tokens.length; i++) {
        address token = _colls.tokens[i];
        if (whitelist.isWrapped(token)) {
            IWAsset(token).updateReward(_borrower, _newOwner, _colls.amounts[i]);
        }
    }
}
```

