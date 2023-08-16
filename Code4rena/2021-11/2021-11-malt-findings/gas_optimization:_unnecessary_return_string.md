## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimization: Unnecessary return string](https://github.com/code-423n4/2021-11-malt-findings/issues/385) 

# Handle

gzeon


# Vulnerability details

## Impact
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/PoolTransferVerification.sol#L61
```
    return (
      maltDataLab.maltPriceAverage(priceLookback) > priceTarget * (10000 - thresholdBps) / 10000,
      "The price of Malt is below peg. Wait for peg to be regained or purchase arbitrage tokens."
    );
```
when the condition is true (which should be the majority of time), the reason string is unnecessary. Only return the string when the condition is false.

