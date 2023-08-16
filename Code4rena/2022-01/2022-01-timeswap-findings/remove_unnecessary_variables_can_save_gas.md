## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Remove unnecessary variables can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/161) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/DateTime.sol#L127-L134

```solidity
    function isValidDate(uint year, uint month, uint day) internal pure returns (bool valid) {
        if (year >= 1970 && month > 0 && month <= 12) {
            uint daysInMonth = _getDaysInMonth(year, month);
            if (day > 0 && day <= daysInMonth) {
                valid = true;
            }
        }
    }
```

The local variable `daysInMonth` is used only once. Making the expression inline can save gas.

### Recommendation

Change to:

```solidity
    function isValidDate(uint year, uint month, uint day) internal pure returns (bool valid) {
        if (year >= 1970 && month > 0 && month <= 12) {
            if (day > 0 && day <= _getDaysInMonth(year, month)) {
                valid = true;
            }
        }
    }
```

Other examples include:

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/Burn.sol#L76-L77

```solidity
        IPair pair = factory.getPair(params.asset, params.collateral);
        require(address(pair) != address(0), 'E501');
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/TimeswapConvenience.sol#L556-L558

```solidity
        IDue collateralizedDebt = natives[asset][collateral][maturity].collateralizedDebt;

        require(msg.sender == address(collateralizedDebt), 'E701');
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/Callback.sol#L62-L63

```solidity
        uint256 _assetReserve = asset.safeBalance();
        require(_assetReserve >= assetReserve + assetIn, 'E304');
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/Callback.sol#L51-L52

```solidity
        uint256 _collateralReserve = collateral.safeBalance();
        require(_collateralReserve >= collateralReserve + collateralIn, 'E305');
```

