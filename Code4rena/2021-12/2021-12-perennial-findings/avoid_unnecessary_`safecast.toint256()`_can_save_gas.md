## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary `SafeCast.toInt256()` can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/30) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L29-L29

```solidity=29
uint256 private _decimalOffset;
```

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L50-L60

```solidity=50{52}
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

`_decimalOffset` is only used at L52。

Therefore `_decimalOffset` can be defined as `int256` and avoid unnecessary `SafeCast.toInt256()`.

