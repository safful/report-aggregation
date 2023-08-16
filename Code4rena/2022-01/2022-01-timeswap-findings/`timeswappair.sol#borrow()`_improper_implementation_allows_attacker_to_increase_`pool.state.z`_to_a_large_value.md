## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`TimeswapPair.sol#borrow()` Improper implementation allows attacker to increase `pool.state.z` to a large value](https://github.com/code-423n4/2022-01-timeswap-findings/issues/162) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, `borrow()` takes a user input value of `zIncrease`, while the actual collateral asset transferred in is calculated at L319, the state of `pool.state.z` still increased by the value of the user's input at L332.

Even though a large number of `zIncrease` means that the user needs to add more collateral, the attacker can use a dust amount `xDecrease` (1 wei for example) so that the total collateral needed is rather small.

Plus, the attacker can always `pay()` the dust amount of loan to get back the rather large amount of collateral added.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L299-L338

```solidity
function borrow(
    uint256 maturity,
    address assetTo,
    address dueTo,
    uint112 xDecrease,
    uint112 yIncrease,
    uint112 zIncrease,
    bytes calldata data
) external override lock returns (uint256 id, Due memory dueOut) {
    require(block.timestamp < maturity, 'E202');
    require(assetTo != address(0) && dueTo != address(0), 'E201');
    require(assetTo != address(this) && dueTo != address(this), 'E204');
    require(xDecrease > 0, 'E205');

    Pool storage pool = pools[maturity];
    require(pool.state.totalLiquidity > 0, 'E206');

    BorrowMath.check(pool.state, xDecrease, yIncrease, zIncrease, fee);

    dueOut.debt = BorrowMath.getDebt(maturity, xDecrease, yIncrease);
    dueOut.collateral = BorrowMath.getCollateral(maturity, pool.state, xDecrease, zIncrease);
    dueOut.startBlock = BlockNumber.get();

    Callback.borrow(collateral, dueOut.collateral, data);

    id = pool.dues[dueTo].insert(dueOut);

    pool.state.reserves.asset -= xDecrease;
    pool.state.reserves.collateral += dueOut.collateral;
    pool.state.totalDebtCreated += dueOut.debt;

    pool.state.x -= xDecrease;
    pool.state.y += yIncrease;
    pool.state.z += zIncrease;

    asset.safeTransfer(assetTo, xDecrease);

    emit Sync(maturity, pool.state.x, pool.state.y, pool.state.z);
    emit Borrow(maturity, msg.sender, assetTo, dueTo, xDecrease, id, dueOut);
}
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/BorrowMath.sol#L62-L79

```solidity
function getCollateral(
    uint256 maturity,
    IPair.State memory state,
    uint112 xDecrease,
    uint112 zIncrease
) internal view returns (uint112 collateralIn) {
    uint256 _collateralIn = maturity;
    _collateralIn -= block.timestamp;
    _collateralIn *= zIncrease;
    _collateralIn = _collateralIn.shiftRightUp(25);
    uint256 minimum = state.z;
    minimum *= xDecrease;
    uint256 denominator = state.x;
    denominator -= xDecrease;
    minimum = minimum.divUp(denominator);
    _collateralIn += minimum;
    collateralIn = _collateralIn.toUint112();
}
```

## PoC

Near the maturity time, the attacker can do the following:

1. `borrow()` a dust amount of assets (`xDecrease` = 1 wei) and increase `pool.state.z` to an extremely large value (20x of previous `state.z` in our tests);
2. `pay()` the loan and get back the collateral;
3. `lend()` a regular amount of `state.x`, get a large amount of insurance token;
4. `burn()` the insurance token and get a large portion of the collateral assets from the defaulted loans.

## Recommendation

Consider making `pair.borrow()` to be `onlyConvenience`, so that `zIncrease` will be a computed value (based on `xDecrease` and current state) rather than a user input value.

