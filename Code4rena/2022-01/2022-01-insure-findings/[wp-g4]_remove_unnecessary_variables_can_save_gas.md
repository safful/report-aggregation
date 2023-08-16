## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [[WP-G4] Remove unnecessary variables can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/216) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L109-L113

```solidity
string memory _name = "InsureDAO-CDS";
string memory _symbol = "iCDS";
uint8 _decimals = IERC20Metadata(_references[0]).decimals();

initializeToken(_name, _symbol, _decimals);
```

The local variable `_name`, `_symbol`, `_decimals` is used only once. Making the expression inline can save gas.

### Recommendation

Change to:

```solidity
initializeToken("InsureDAO-CDS", "iCDS", IERC20Metadata(_references[0]).decimals());
```

Other examples include:

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L189-L190

```solidity
uint256 _balance = balanceOf(msg.sender);
require(_balance >= _amount, "ERROR: REQUEST_EXCEED_BALANCE");
```

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/CDSTemplate.sol#L257-L257

```solidity
uint256 _surplusAttribution = surplusPool;
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

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/IndexTemplate.sol#L456-L463

```solidity
uint256 _shortage;
if (totalLiquidity() < _amount) {
    //Insolvency case
    _shortage = _amount - _value;
    uint256 _cds = ICDSTemplate(registry.getCDS(address(this)))
        .compensate(_shortage);
    _compensated = _value + _cds;
}
```

`_shortage` and `_cds`.

