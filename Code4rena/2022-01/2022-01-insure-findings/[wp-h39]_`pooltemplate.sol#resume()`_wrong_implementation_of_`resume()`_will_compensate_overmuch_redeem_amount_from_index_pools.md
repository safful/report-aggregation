## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H39] `PoolTemplate.sol#resume()` Wrong implementation of `resume()` will compensate overmuch redeem amount from index pools](https://github.com/code-423n4/2022-01-insure-findings/issues/283) 

# Handle

WatchPug


# Vulnerability details

## Root Cause

Wrong arithmetic.

---

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L700-L717

```solidity
uint256 _deductionFromIndex = (_debt * _totalCredit * MAGIC_SCALE_1E6) /
            totalLiquidity();
    uint256 _actualDeduction;
    for (uint256 i = 0; i < indexList.length; i++) {
        address _index = indexList[i];
        uint256 _credit = indicies[_index].credit;
        if (_credit > 0) {
            uint256 _shareOfIndex = (_credit * MAGIC_SCALE_1E6) /
                _totalCredit;
            uint256 _redeemAmount = _divCeil(
                _deductionFromIndex,
                _shareOfIndex
            );
            _actualDeduction += IIndexTemplate(_index).compensate(
                _redeemAmount
            );
        }
    }
```


### PoC

- totalLiquidity = 200,000* 10**18;
- totalCredit = 100,000 * 10**18;
- debt = 10,000 * 10**18;

- [Index Pool 1] Credit = 20,000 * 10**18;
- [Index Pool 2] Credit = 30,000 * 10**18;

```
uint256 _deductionFromIndex = (_debt * _totalCredit * MAGIC_SCALE_1E6) /
            totalLiquidity();
// _deductionFromIndex = 10,000 * 10**6 * 10**18;

```

[Index Pool 1]:

```
uint256 _shareOfIndex = (_credit * MAGIC_SCALE_1E6) / _totalCredit;  
//  _shareOfIndex = 200000

uint256 _redeemAmount = _divCeil(
    _deductionFromIndex,
    _shareOfIndex
);

// _redeemAmount = 25,000 * 10**18;
```

[Index Pool 2]:

```
uint256 _shareOfIndex = (_credit * MAGIC_SCALE_1E6) / _totalCredit;  
//  _shareOfIndex = 300000

uint256 _redeemAmount = _divCeil(
    _deductionFromIndex,
    _shareOfIndex
);

// _redeemAmount = 16666666666666666666667 (~ 16,666 * 10**18)
```

In most cases, the transaction will revet on underflow at:
```
uint256 _shortage = _deductionFromIndex /
            MAGIC_SCALE_1E6 -
            _actualDeduction;
```

In some cases, specific pools will be liable for unfair compensation:

If the CSD is empty, `Index Pool 1` only have `6,000 * 10**18` and `Index Pool 2` only have `4,000 * 10**18`, the `_actualDeduction` will be `10,000 * 10**18`, `_deductionFromPool` will be `0`.


`Index Pool 1` should only pay `1,000 * 10**18`, but actually paid `6,000 * 10**18`, the LPs of `Index Pool 1` now suffer funds loss.

### Recommendation

Change to:

```solidity
uint256 _deductionFromIndex = (_debt * _totalCredit * MAGIC_SCALE_1E6) / totalLiquidity();
uint256 _actualDeduction;
for (uint256 i = 0; i < indexList.length; i++) {
    address _index = indexList[i];
    uint256 _credit = indicies[_index].credit;
    if (_credit > 0) {
        uint256 _shareOfIndex = (_credit * MAGIC_SCALE_1E6) /
            _totalCredit;
        uint256 _redeemAmount = _divCeil(
            _deductionFromIndex * _shareOfIndex,
            MAGIC_SCALE_1E6 * MAGIC_SCALE_1E6
        );
        _actualDeduction += IIndexTemplate(_index).compensate(
            _redeemAmount
        );
    }
}
```

