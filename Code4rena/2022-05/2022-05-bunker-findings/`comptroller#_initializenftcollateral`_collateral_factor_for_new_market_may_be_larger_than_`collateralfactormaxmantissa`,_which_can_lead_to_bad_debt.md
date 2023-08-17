## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [`Comptroller#_initializeNftCollateral` Collateral Factor for new market may be larger than `collateralFactorMaxMantissa`, which can lead to bad debt](https://github.com/code-423n4/2022-05-bunker-findings/issues/79) 

# Lines of code

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/Comptroller.sol#L1337-L1359


# Vulnerability details

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/Comptroller.sol#L1337-L1359

```solidity
function _initializeNftCollateral(CNftInterface cNft, NftPriceOracle _nftOracle, uint256 _collateralFactorMantissa) external returns (uint) {
    require(address(nftMarket) == address(0), "nft collateral already initialized");
    require(address(cNft) != address(0), "cannot initialize nft market to the 0 address");

    if (msg.sender != admin) {
        return fail(Error.UNAUTHORIZED, FailureInfo.SUPPORT_MARKET_OWNER_CHECK);
    }

    if (markets[address(cNft)].isListed) {
        return fail(Error.MARKET_ALREADY_LISTED, FailureInfo.SUPPORT_MARKET_EXISTS);
    }

    cNft.isCNft(); // Sanity check to make sure its really a cNFT.

    nftMarket = cNft;
    nftOracle = _nftOracle;

    // Note that isComped is not in active use anymore
    markets[address(cNft)] = Market({isListed: true, isComped: false, collateralFactorMantissa: _collateralFactorMantissa});

    // We do not support borrowing NFTs.
    borrowGuardianPaused[address(cNft)] = false;
}
```

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/Comptroller.sol#L80-L81

```solidity
// No collateralFactorMantissa may exceed this value
uint internal constant collateralFactorMaxMantissa = 0.9e18; // 0.9
```


There is a `collateralFactorMaxMantissa` which limits the collateral factor to be always lower than 0.9.

Per the comment:

> No collateralFactorMantissa may exceed this value

However, in `_initializeNftCollateral()`, the `_collateralFactorMantissa` is set without any check, which means it can be set to a value > 0.9 or even > 1.

As a result, the borrowers may deliberately choose not to repay as their collateral may not be worth the debt in the first place. This can accumulate bad debt in the whole system.

### Recommendation

Change to:

```solidity
// Check collateral factor <= 0.9
Exp memory highLimit = Exp({mantissa: collateralFactorMaxMantissa});
if (lessThanExp(highLimit, _collateralFactorMantissa)) {
    return fail(Error.INVALID_COLLATERAL_FACTOR, FailureInfo.SET_COLLATERAL_FACTOR_VALIDATION);
}

markets[address(cNft)] = Market({isListed: true, isComped: false, collateralFactorMantissa: _collateralFactorMantissa});
```

