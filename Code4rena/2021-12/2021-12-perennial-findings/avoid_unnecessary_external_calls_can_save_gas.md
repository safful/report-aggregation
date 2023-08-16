## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary external calls can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/26) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L50-L60

```solidity=50
function sync() public {
    (, int256 feedPrice, , uint256 timestamp, ) = feed.latestRoundData();
    Fixed18 price = Fixed18Lib.ratio(feedPrice, SafeCast.toInt256(_decimalOffset));

    if (priceAtVersion.length == 0 || timestamp > timestampAtVersion[currentVersion()] + minDelay) {
        priceAtVersion.push(price);
        timestampAtVersion.push(timestamp);

        emit Version(currentVersion(), timestamp, price);
    }
}
```

If `block.timestamp - timestampAtVersion[currentVersion()] < minDelay`, there is no need to call `feed.latestRoundData()`.

### Recommendation

Change to:

```solidity=50
function sync() public {
    if (priceAtVersion.length == 0 || block.timestamp - timestampAtVersion[currentVersion()] >= minDelay ) {
        (, int256 feedPrice, , uint256 timestamp, ) = feed.latestRoundData();
        Fixed18 price = Fixed18Lib.ratio(feedPrice, SafeCast.toInt256(_decimalOffset));

        if (priceAtVersion.length == 0 || timestamp > timestampAtVersion[currentVersion()] + minDelay) {
            priceAtVersion.push(price);
            timestampAtVersion.push(timestamp);

            emit Version(currentVersion(), timestamp, price);
        }
    }
}
```

