## Tags

- bug
- sponsor confirmed
- 3 (High Risk)
- resolved

# [The formula of number of prizes for a degree is wrong](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/33) 

# Handle

WatchPug


# Vulnerability details

The formula of the number of prizes for a degree per the document: https://v4.docs.pooltogether.com/protocol/concepts/prize-distribution/#splitting-the-prizes is:
```
Number of prizes for a degree = (2^bit range)^degree - (2^bit range)^(degree-1) - (2^bit range)^(degree-2) - ...
```
Should be changed to:
```
Number of prizes for a degree = (2^bit range)^degree - (2^bit range)^(degree-1)
```
or
```
Number of prizes for a degree = 2^(bit range * degree) - 2^(bit range * (degree-1))
```

### Impact

Per the document:

> prize for a degree = total prize * degree percentage / number of prizes for a degree

Due to the miscalculation of `number of prizes for a degree`, it will be smaller than expected, as a result, `prize for a degree` will be larger than expected. Making the protocol giving out more prizes than designed.

### Proof

> We will use `f(bitRange, degree)` to represent `numberOfPrizesForDegree(bitRangeSize, degree)`.

#### Proof: (method 1)

```tex
2 ^ {bitRange \times n} = f(bitRange, n) + f(bitRange, n-1) + f(bitRange, n-2) + ... + f(bitRange, 1) + f(bitRange, 0)
f(bitRange, n) = 2 ^ {bitRange \times n} - ( f(bitRange, n-1) + f(bitRange, n-2) + ... + f(bitRange, 1) + f(bitRange, 0) )
f(bitRange, n) = 2 ^ {bitRange \times n} - f(bitRange, n-1) - ( f(bitRange, n-2) + ... + f(bitRange, 1) + f(bitRange, 0) )

Because:

2 ^ {bitRange \times (n-1)} = f(bitRange, n-1) + f(bitRange, n-2) + ... + f(bitRange, 1) + f(bitRange, 0)
2 ^ {bitRange \times (n-1)} - f(bitRange, n-1) = f(bitRange, n-2) + ... + f(bitRange, 1) + f(bitRange, 0)

Therefore:

f(bitRange, n) = 2 ^ {bitRange \times n} - f(bitRange, n-1) - ( 2 ^ {bitRange \times (n-1)} - f(bitRange, n-1) )
f(bitRange, n) = 2 ^ {bitRange \times n} - f(bitRange, n-1) - 2 ^ {bitRange \times (n-1)} + f(bitRange, n-1)
f(bitRange, n) = 2 ^ {bitRange \times n} - 2 ^ {bitRange \times (n-1)}
```

Because `2^x = 1 << x`

Therefore, when `n > 0`:

```
f(bitRange, n) = ( 1 << bitRange * n ) - ( 1 << bitRange * (n - 1) )
```

QED.

#### Proof: (method 2)

By definition, `degree n` is constructed by 3 chunks:

-  The first N numbers, must equal the matching numbers. Number of possible values: `1`;
-  The N-th number, must not equal the N-th matching number. Number of possible values: `2^bitRange - 1`
-  From N (not include) until the end. Number of possible values: `2 ^ (bitRange * (n-1))`

Therefore, total `numberOfPrizesForDegree` will be:

```tex
f(bitRange, n) = (2 ^ {bitRange} - 1) \times 2 ^ {bitRange \times (n - 1)}
f(bitRange, n) = 2 ^ {bitRange} \times 2 ^ {bitRange \times (n - 1)} - 2 ^ {bitRange \times (n - 1)}
f(bitRange, n) = 2 ^ {bitRange + bitRange \times (n - 1)} - 2 ^ {bitRange \times (n - 1)}
f(bitRange, n) = 2 ^ {bitRange + bitRange \times n - bitRange} - 2 ^ {bitRange \times (n - 1)}
f(bitRange, n) = 2 ^ {bitRange \times n} - 2 ^ {bitRange \times (n - 1)}
```

QED.

### Recommendation

https://github.com/pooltogether/v4-core/blob/055335bf9b09e3f4bbe11a788710dd04d827bf37/contracts/DrawCalculator.sol#L423-L431


```solidity=412{423-431}
/**
    * @notice Calculates the number of prizes for a given prizeDistributionIndex
    * @param _bitRangeSize Bit range size for Draw
    * @param _prizeTierIndex Index of the prize tier array to calculate
    * @return returns the fraction of the total prize (base 1e18)
    */
function _numberOfPrizesForIndex(uint8 _bitRangeSize, uint256 _prizeTierIndex)
    internal
    pure
    returns (uint256)
{
    uint256 bitRangeDecimal = 2**uint256(_bitRangeSize);
    uint256 numberOfPrizesForIndex = bitRangeDecimal**_prizeTierIndex;

    while (_prizeTierIndex > 0) {
        numberOfPrizesForIndex -= bitRangeDecimal**(_prizeTierIndex - 1);
        _prizeTierIndex--;
    }

    return numberOfPrizesForIndex;
}
```

L423-431 should change to:

```solidity
if (_prizeTierIndex > 0) {
    return ( 1 << _bitRangeSize * _prizeTierIndex ) - ( 1 << _bitRangeSize * (_prizeTierIndex - 1) );
} else {
    return 1;
}
```

BTW, the comment on L416 is wrong:

- seems like it's copied from _calculatePrizeTierFraction()
- plus, it's not base 1e18 but base 1e9

