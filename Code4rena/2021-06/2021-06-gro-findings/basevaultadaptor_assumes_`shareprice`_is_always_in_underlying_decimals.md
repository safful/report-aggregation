## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [BaseVaultAdaptor assumes `sharePrice` is always in underlying decimals](https://github.com/code-423n4/2021-06-gro-findings/issues/114) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The two `BaseVaultAdaptor.calculateShare` functions computes `share = amount.mul(uint256(10)**decimals).div(sharePrice)`

```solidity
uint256 sharePrice = _getVaultSharePrice();
// amount is in "token" decimals, share should be in "vault" decimals
share = amount.mul(uint256(10)**decimals).div(sharePrice);
```

This assumes that the `sharePrice` is always in _token_ decimals and that _token_ decimals is the same as _vault_ decimals.

This both happens to be the case for Yearn vaults, but will not necessarily be the case for other protocols.
As this functionality is in the `BaseVaultAdaptor` and not in the specific `VaultAdaptorYearnV2_032`, consider generalizing the conversion.

## Impact
Integrating a token where the token or price is reported in a different precision will lead to potential losses as more shares are computed.

## Recommended Mitigation Steps
The conversion seems highly protocol specific, `calculateShare` should be an abstract function like `_getVaultSharePrice`, that is implemented in the specific adaptors.

