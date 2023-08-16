## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [In `CreditLine#_borrowTokensToLiquidate`, oracle is used wrong way](https://github.com/code-423n4/2021-12-sublime-findings/issues/155) 

# Handle

0x0x0x


# Vulnerability details

Current implementation to get the price is as follows:

`(uint256 _ratioOfPrices, uint256 _decimals) = IPriceOracle(priceOracle).getLatestPrice(_borrowAsset, _collateralAsset);`

[https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L1050](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L1050)

But it should not consult `borrowToken / collateralToken`, rather it should consult the inverse of this result. As a consequence, in `liquidate` the liquidator/lender can lose/gain funds as a result of this miscalculation.

## Mitigation step

Replace it with

`(uint256 _ratioOfPrices, uint256 _decimals) = IPriceOracle(priceOracle).getLatestPrice(_collateralAsset, _borrowAsset);`

