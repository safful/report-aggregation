## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`YearnVault.sol#pull()` will most certainly fail](https://github.com/code-423n4/2021-12-mellow-findings/issues/121) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/test_brownie/contracts/YearnVault.sol#L84-L101

```solidity=84
    for (uint256 i = 0; i < _yTokens.length; i++) {
        if (tokenAmounts[i] == 0) {
            continue;
        }

        IYearnVault yToken = IYearnVault(_yTokens[i]);
        uint256 yTokenAmount = ((tokenAmounts[i] * (10**yToken.decimals())) / yToken.pricePerShare());
        uint256 balance = yToken.balanceOf(address(this));
        if (yTokenAmount > balance) {
            yTokenAmount = balance;
        }
        if (yTokenAmount == 0) {
            continue;
        }
        yToken.withdraw(yTokenAmount, to, maxLoss);
        (tokenAmounts[i], address(this));
    }
    actualTokenAmounts = tokenAmounts;
```

The actual token withdrew from `yToken.withdraw()` will most certainly be less than the `tokenAmounts[i]`, due to precision loss in the calculation of `yTokenAmount`.

As a result, `IERC20(_vaultTokens[i]).safeTransfer(to, actualTokenAmounts[i]);` in `LpIssuer.sol#withdraw()` will revert due to insufficant balance.

 ### Recommendation

Change to:

```solidity=98
tokenAmounts[i] = yToken.withdraw(yTokenAmount, to, maxLoss);
```

