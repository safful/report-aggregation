## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Yearn token <> shares conversion decimal issue](https://github.com/code-423n4/2021-12-sublime-findings/issues/134) 

# Handle

cmichel


# Vulnerability details

The yearn strategy `YearnYield` converts shares to tokens by doing `pricePerFullShare * shares / 1e18`:

```
function getTokensForShares(uint256 shares, address asset) public view override returns (uint256 amount) {
    if (shares == 0) return 0;
    // @audit should divided by vaultDecimals 
    amount = IyVault(liquidityToken[asset]).getPricePerFullShare().mul(shares).div(1e18);
}
```

But Yearn's `getPricePerFullShare` seems to be [in `vault.decimals()` precision](https://github.com/yearn/yearn-vaults/blob/03b42dacacec2c5e93af9bf3151da364d333c222/contracts/Vault.vy#L1147), i.e., it should convert it as `pricePerFullShare * shares / (10 ** vault.decimals())`.
The vault decimals are the same [as the underlying token decimals](https://github.com/yearn/yearn-vaults/blob/03b42dacacec2c5e93af9bf3151da364d333c222/contracts/Vault.vy#L295-L296)

## Impact
The token and shares conversions do not work correctly for underlying tokens that do not have 18 decimals.
Too much or too little might be paid out leading to a loss for either the protocol or user.

## Recommended Mitigation Steps
Divide by `10**vault.decimals()` instead of `1e18` in `getTokensForShares`.
Apply a similar fix in `getSharesForTokens`.

