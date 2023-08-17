## Tags

- bug
- 2 (Med Risk)
- satisfactory
- sponsor confirmed
- selected for report
- M-12

# [ Users could get some `DOLA` even if their are on liquidation position](https://github.com/code-423n4/2022-10-inverse-findings/issues/419) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L566


# Vulnerability details

## Impact
Users abels to invoke `forceReplenish()` when they are on liquidation position

## Proof of Concept
On `Market.sol` ==>  `forceReplenish()`
On this line 
```
uint collateralValue = getCollateralValueInternal(user);
```

`getCollateralValueInternal(user)` only return the value of the collateral 
```
    function getCollateralValueInternal(address user) internal returns (uint) {
        IEscrow escrow = predictEscrow(user);
        uint collateralBalance = escrow.balance();
        return collateralBalance * oracle.getPrice(address(collateral), collateralFactorBps) / 1 ether; 
```
So if the user have 1.5 wETH at the price of  1 ETH = 1600 USD
It will return `1.5 * 1600` and this value is the real value we can’t just check it directly with the debt like this 
```
 require(collateralValue >= debts[user], "Exceeded collateral value");
```
This is no longer `over collateralized` protocol 
The value needs to be multiplied by `collateralFactorBps / 10000`
-  So depending on the value of `collateralFactorBps` and `liquidationFactorBps` the user could be in the liquidation position but he is able to invoke `forceReplenish()` to cover all their `dueTokensAccrued[user]` on `DBR.sol` and get more `DOLA`
-  or it will lead a healthy debt to be in the liquidation position after invoking `forceReplenish()`
- 

## Recommended Mitigation Steps
Use `getCreditLimitInternal()` rather than `getCollateralValueInternal()`.

