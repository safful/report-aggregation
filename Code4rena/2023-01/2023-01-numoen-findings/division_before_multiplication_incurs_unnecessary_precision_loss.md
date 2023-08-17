## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-06

# [Division before multiplication incurs unnecessary precision loss](https://github.com/code-423n4/2023-01-numoen-findings/issues/45) 

# Lines of code

https://github.com/code-423n4/2023-01-numoen/blob/2ad9a73d793ea23a25a381faadc86ae0c8cb5913/src/core/Pair.sol#L56
https://github.com/code-423n4/2023-01-numoen/blob/2ad9a73d793ea23a25a381faadc86ae0c8cb5913/src/core/Pair.sol#L57
https://github.com/code-423n4/2023-01-numoen/blob/2ad9a73d793ea23a25a381faadc86ae0c8cb5913/src/core/Lendgine.sol#L252


# Vulnerability details

## Impact

Division before multipilication incurs uncessary precision loss

## Proof of Concept

In the current codebase, FullMath.mulDiv is used, the function takes three parameter, 

basically FullMath.mulDIv(a, b, c) means a * b / c

Then there are some operation which that incurs unnecessary precision loss because of division before multiplcation.

When accuring interest, the code belows:

```solidity
  /// @notice Helper function for accruing lendgine interest
  function _accrueInterest() private {
    if (totalSupply == 0 || totalLiquidityBorrowed == 0) {
      lastUpdate = block.timestamp;
      return;
    }

    uint256 timeElapsed = block.timestamp - lastUpdate;
    if (timeElapsed == 0) return;

    uint256 _totalLiquidityBorrowed = totalLiquidityBorrowed; // SLOAD
    uint256 totalLiquiditySupplied = totalLiquidity + _totalLiquidityBorrowed; // SLOAD

    uint256 borrowRate = getBorrowRate(_totalLiquidityBorrowed, totalLiquiditySupplied);

    uint256 dilutionLPRequested = (FullMath.mulDiv(borrowRate, _totalLiquidityBorrowed, 1e18) * timeElapsed) / 365 days;
    uint256 dilutionLP = dilutionLPRequested > _totalLiquidityBorrowed ? _totalLiquidityBorrowed : dilutionLPRequested;
    uint256 dilutionSpeculative = convertLiquidityToCollateral(dilutionLP);

    totalLiquidityBorrowed = _totalLiquidityBorrowed - dilutionLP;
    rewardPerPositionStored += FullMath.mulDiv(dilutionSpeculative, 1e18, totalPositionSize);
    lastUpdate = block.timestamp;

    emit AccrueInterest(timeElapsed, dilutionSpeculative, dilutionLP);
  }
```

note the line:

```solidity
 uint256 dilutionLPRequested = (FullMath.mulDiv(borrowRate, _totalLiquidityBorrowed, 1e18) * timeElapsed) / 365 days;
```

This basically equals to dilutionLPRequested = (borrowRate * totalLiquidityBorrowed / 1e18 * timeElapsed) / 365 days

the first part of division can greatly truncate the value borrowRate * totalLiquidityBorrowed / 1e18, the totalLiquidityBorrowed should normalized and scaled by token preciision when adding liqudiity instead of division by 1e18 here.

Same preicision loss happens when computng the invariant

```solidity
  /// @inheritdoc IPair
  function invariant(uint256 amount0, uint256 amount1, uint256 liquidity) public view override returns (bool) {
    if (liquidity == 0) return (amount0 == 0 && amount1 == 0);

    uint256 scale0 = FullMath.mulDiv(amount0, 1e18, liquidity) * token0Scale;
    uint256 scale1 = FullMath.mulDiv(amount1, 1e18, liquidity) * token1Scale;
```

scale0 = (amount0 * 1e18 / liqudiity) * token0Scale
scale1 = (amount1 * 1e18 / liqudiity) * token1Scale

whereas the amount0 and amount1 should be first be normalized by token0Scale and token1Scale and then divided by liquidity at last. If the liquidity is a larger number  amount0 * 1e18 / liqudiity is already truncated to 0.

## Tools Used

Manual Review

## Recommended Mitigation Steps

We recommend the protocol avoid divison before multiplication and always perform division operation at last.