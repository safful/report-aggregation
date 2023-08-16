## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-10-tracer-findings/issues/27) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

- `LeveragedPool.sol#intervalPassed()`

    https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L259-L261

    ```solidity
        function intervalPassed() public view override returns (bool) {
            return block.timestamp >= lastPriceTimestamp + updateInterval;
        }
    ```

    `lastPriceTimestamp + updateInterval` will never overlow.

- `LeveragedPool.sol#executePriceChange()`

    https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L175-L204

    ```solidity
    emit PoolRebalance(
        int256(newShortBalance) - int256(_shortBalance),
        int256(newLongBalance) - int256(_longBalance)
    );
    ```

    `int256(newShortBalance) - int256(_shortBalance)` and `int256(newLongBalance) - int256(_longBalance)` will never underflow.

