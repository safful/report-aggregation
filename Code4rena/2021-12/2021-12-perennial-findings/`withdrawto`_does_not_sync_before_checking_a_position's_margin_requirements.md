## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [`withdrawTo` Does Not Sync Before Checking A Position's Margin Requirements](https://github.com/code-423n4/2021-12-perennial-findings/issues/74) 

# Handle

leastwood


# Vulnerability details

## Impact

The `maintenanceInvariant` modifier in `Collateral` aims to check if a user meets the margin requirements to withdraw collateral by checking its current and next maintenance. `maintenanceInvariant` inevitably calls `AccountPosition.maintenance` which uses the oracle's price to calculate the margin requirements for a given position. Hence, if the oracle has not synced in a long time, `maintenanceInvariant` may end up utilising an outdated price for a withdrawal. This may allow a user to withdraw collateral on an undercollaterized position.

## Proof of Concept

https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/collateral/Collateral.sol#L67-L76
```
function withdrawTo(address account, IProduct product, UFixed18 amount)
notPaused
collateralInvariant(msg.sender, product)
maintenanceInvariant(msg.sender, product)
external {
    _products[product].debitAccount(msg.sender, amount);
    token.push(account, amount);

    emit Withdrawal(msg.sender, product, amount);
}
```

https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/collateral/Collateral.sol#L233-L241
```
modifier maintenanceInvariant(address account, IProduct product) {
    _;

    UFixed18 maintenance = product.maintenance(account);
    UFixed18 maintenanceNext = product.maintenanceNext(account);

    if (UFixed18Lib.max(maintenance, maintenanceNext).gt(collateral(account, product)))
        revert CollateralInsufficientCollateralError();
}
```

https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/product/types/position/AccountPosition.sol#L71-L75
```
    function maintenanceInternal(Position memory position, IProductProvider provider) private view returns (UFixed18) {
        Fixed18 oraclePrice = provider.priceAtVersion(provider.currentVersion());
        UFixed18 notionalMax = Fixed18Lib.from(position.max()).mul(oraclePrice).abs();
        return notionalMax.mul(provider.maintenance());
    }
```

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider adding `settleForAccount(msg.sender)` to the `Collateral.withdrawTo` function to ensure the most up to date oracle price is used when assessing an account's margin requirements.

