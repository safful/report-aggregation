## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Code Style: consistency](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/228) 

# Handle

WatchPug


# Vulnerability details

The parameter names of event `RampTargetPrice` should be the same as the struct `TargetPrice` for consistency.

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L61-L78

```solidity=61
event RampTargetPrice(
    uint256 oldTargetPrice,
    uint256 newTargetPrice,
    uint256 initialTime,
    uint256 futureTime
);
```

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L117-L124

```solidity=117{120-121}
struct TargetPrice {
    uint256 initialTargetPrice;
    uint256 futureTargetPrice;
    uint256 initialTargetPriceTime;
    uint256 futureTargetPriceTime;
    
    uint256[2] originalPrecisionMultipliers;
}
```

### Recommendation

Consider changing to:

```solidity
event RampTargetPrice(
    uint256 oldTargetPrice,
    uint256 newTargetPrice,
    uint256 initialTargetPriceTime,
    uint256 futureTargetPriceTime
);
```

