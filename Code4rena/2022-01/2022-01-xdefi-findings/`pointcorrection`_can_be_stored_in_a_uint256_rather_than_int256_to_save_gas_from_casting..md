## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`pointCorrection` can be stored in a uint256 rather than int256 to save gas from casting.](https://github.com/code-423n4/2022-01-xdefi-findings/issues/87) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Extra gas costs from unnecessary casting.

## Proof of Concept

`pointsCorrection` is stored as a int256 variable.

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/interfaces/IXDEFIDistribution.sol#L15

However we can see that this variable is always negative (`_pointsPerUnit` and  `units` are both positive)

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L277

The only usage of `pointsCorrection` is in the `_withdrawableGiven` function as shown below.

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L341-L342

```
(
  _toUint256Safe(
      _toInt256Safe(_pointsPerUnit * uint256(units_)) +
      pointsCorrection_
  ) / _pointsMultiplier
) + uint256(depositedXDEFI_);
```

`pointsCorrection` is set to `_pointsPerUnit * uint256(units_)` when locking and `_pointsPerUnit` only increases so we can safely store `pointsCorrection as a positive uint256 (note this is an assumption of the original code as well) and simplify the above expression.

```
// notice the sign change before `pointsCorrection_`
(_pointsPerUnit * uint256(units_) - pointsCorrection_) / _pointsMultiplier + uint256(depositedXDEFI_);
```

We can then remove a significant amount of casting along with the associated costs.

## Recommended Mitigation Steps

store `pointsCorrection` in a uint256 and subtract rather than add.

