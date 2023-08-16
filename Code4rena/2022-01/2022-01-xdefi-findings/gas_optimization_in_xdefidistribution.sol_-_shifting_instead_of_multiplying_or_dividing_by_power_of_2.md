## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in XDEFIDistribution.sol - shifting instead of multiplying or dividing by power of 2](https://github.com/code-423n4/2022-01-xdefi-findings/issues/122) 

# Handle

OriDabush


# Vulnerability details

## XDEFIDistribution.sol lines 151, 338-344
Instead of multiplying by _pointsMultiplier, which is 2 ** 128, it is more efficient to shift by 128 (x * (2 ** 128) = x << 128), same for dividing (x / (2 ** 128) = x >> 128)
```sol
// line 151 - old
_pointsPerUnit += ((newXDEFI * _pointsMultiplier) / totalUnitsCached);

// line 151 - new
_pointsPerUnit += ((newXDEFI << 128) / totalUnitsCached);


// lines 338-344 - old
return
(
    _toUint256Safe(
        _toInt256Safe(_pointsPerUnit * uint256(units_)) +
        pointsCorrection_
    ) / _pointsMultiplier
) + uint256(depositedXDEFI_);

// lines 338-344 - new
return
(
    _toUint256Safe(
        _toInt256Safe(_pointsPerUnit * uint256(units_)) +
        pointsCorrection_
    ) >> 128
) + uint256(depositedXDEFI_);
```

