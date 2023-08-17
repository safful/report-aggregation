## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [Moving average precision is lost](https://github.com/code-423n4/2022-08-olympus-findings/issues/483) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/modules/PRICE.sol#L134-L139


# Vulnerability details

Now the precision is lost in moving average calculations as the difference is calculated separately and added each time, while it typically can be small enough to lose precision in the division involved.

For example, `10000` moves of `990` size, `numObservations = 1000`. This will yield `0` on each update, while must yield `9900` increase in the moving average.

## Proof of Concept

Moving average is calculated with the addition of the difference:

https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/modules/PRICE.sol#L134-L139

```solidity
        // Calculate new moving average
        if (currentPrice > earliestPrice) {
            _movingAverage += (currentPrice - earliestPrice) / numObs;
        } else {
            _movingAverage -= (earliestPrice - currentPrice) / numObs;
        }
```

`/ numObs` can lose precision as `currentPrice - earliestPrice` is usually small.

It is returned on request as is:

https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/modules/PRICE.sol#L189-L193

```solidity
    /// @notice Get the moving average of OHM in the Reserve asset over the defined window (see movingAverageDuration and observationFrequency).
    function getMovingAverage() external view returns (uint256) {
        if (!initialized) revert Price_NotInitialized();
        return _movingAverage;
    }
```

## Recommended Mitigation Steps

Consider storing the cumulative `sum`, while returning `sum / numObs` on request:

https://github.com/code-423n4/2022-08-olympus/blob/2a0b515012b4a40076f6eac487f7816aafb8724a/src/modules/PRICE.sol#L189-L193

```solidity
    /// @notice Get the moving average of OHM in the Reserve asset over the defined window (see movingAverageDuration and observationFrequency).
    function getMovingAverage() external view returns (uint256) {
        if (!initialized) revert Price_NotInitialized();
-       return _movingAverage;
+       return _movingAverage / numObservations;
    }
```

