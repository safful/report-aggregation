## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary checked arithmetic in for loops](https://github.com/code-423n4/2022-01-timeswap-findings/issues/170) 

# Handle

WatchPug


# Vulnerability details

There is no risk of overflow caused by increamenting the iteration index in for loops (the `i++` in for `for (uint256 i; i < ids.length; i++)`).

Increments perform overflow checks that are not necessary in this case.

### Recommendation

Surround the increment expressions with an `unchecked { ... }` block to avoid the default overflow checks. For example, change the for loop:


https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/PayMath.sol#L21-L33

```solidity
for (uint256 i; i < ids.length; i++) {
    IPair.Due memory due = pair.dueOf(maturity, address(collateralizedDebt), ids[i]);

    if (assetsIn[i] > due.debt) assetsIn[i] = due.debt;
    if (msg.sender == collateralizedDebt.ownerOf(ids[i])) {
        uint256 _collateralOut = due.collateral;
        if (due.debt > 0) {
            _collateralOut *= assetsIn[i];
            _collateralOut /= due.debt;
        }
        collateralsOut[i] = _collateralOut.toUint112();
    }
}
```

to:

```solidity
for (uint256 i; i < ids.length;) {
    IPair.Due memory due = pair.dueOf(maturity, address(collateralizedDebt), ids[i]);

    if (assetsIn[i] > due.debt) assetsIn[i] = due.debt;
    if (msg.sender == collateralizedDebt.ownerOf(ids[i])) {
        uint256 _collateralOut = due.collateral;
        if (due.debt > 0) {
            _collateralOut *= assetsIn[i];
            _collateralOut /= due.debt;
        }
        collateralsOut[i] = _collateralOut.toUint112();
    }

    unchecked { ++i; }
}
```


