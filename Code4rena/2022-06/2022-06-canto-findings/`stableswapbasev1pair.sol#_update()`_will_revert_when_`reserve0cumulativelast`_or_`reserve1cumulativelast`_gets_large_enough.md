## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [`stableswap/BaseV1Pair.sol#_update()` will revert when `reserve0CumulativeLast` or `reserve1CumulativeLast` gets large enough](https://github.com/code-423n4/2022-06-canto-findings/issues/163) 

# Lines of code

https://github.com/Plex-Engineer/stableswap/blob/0dd7ac65d923bb7462c47f6d56b564af34b34118/contracts/BaseV1-core.sol#L154-L171


# Vulnerability details

```solidity
function _update(uint balance0, uint balance1, uint _reserve0, uint _reserve1) internal {
    uint blockTimestamp = block.timestamp;
    uint timeElapsed = blockTimestamp - blockTimestampLast; // overflow is desired
    if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
        reserve0CumulativeLast += _reserve0 * timeElapsed;
        reserve1CumulativeLast += _reserve1 * timeElapsed;
    }

    Observation memory _point = lastObservation();
    timeElapsed = blockTimestamp - _point.timestamp; // compare the last observation with current timestamp, if greater than 30 minutes, record a new event
    if (timeElapsed > periodSize) {
        observations.push(Observation(blockTimestamp, reserve0CumulativeLast, reserve1CumulativeLast));
    }
    reserve0 = balance0;
    reserve1 = balance1;
    blockTimestampLast = blockTimestamp;
    emit Sync(reserve0, reserve1);
}
```

This was forked from Uniswap v2's `update()`:

https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L72-L81

```solidity=L72
// update reserves and, on the first call per block, price accumulators
function _update(uint balance0, uint balance1, uint112 _reserve0, uint112 _reserve1) private {
    require(balance0 <= uint112(-1) && balance1 <= uint112(-1), 'UniswapV2: OVERFLOW');
    uint32 blockTimestamp = uint32(block.timestamp % 2**32);
    uint32 timeElapsed = blockTimestamp - blockTimestampLast; // overflow is desired
    if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
        // * never overflows, and + overflow is desired
        price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
        price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
    }
```

UniswapV2's Pair is using Solidity 0.5.16, in which the arithmetic operations will overflow/underflow without revert.

As the solidity version used in the current implementation of `BaseV1Pair.sol` is `0.8.11`, and there are some breaking changes in Solidity v0.8.0, including:

> Arithmetic operations revert on underflow and overflow. 

Ref: https://docs.soliditylang.org/en/v0.8.11/080-breaking-changes.html#silent-changes-of-the-semantics

When updating `reserve0CumulativeLast` and `reserve1CumulativeLast` in `BaseV1Pair.sol`, overflow and underflow are desired as per the comment.

However, the intended overflow only works for solidity < `0.8.0` by default. If overflow and underflow are desired, then the math should be put into an `unchecked` block. Otherwise, the transaction will revert.

### Impact

Since the overflow is desired in the original version, and it's broken because of using Solidity version >0.8. The `BaseV1Pair` contract will break when the desired overflow happens, which will be sooner or later depending on the decimals of the tokens and trading volume.

### Recommendation

Change to:

```solidity
unchecked {
    uint timeElapsed = blockTimestamp - blockTimestampLast;
    if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
        reserve0CumulativeLast += _reserve0 * timeElapsed;
        reserve1CumulativeLast += _reserve1 * timeElapsed;
    }
}
```

