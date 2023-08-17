## Tags

- bug
- 2 (Med Risk)
- satisfactory
- sponsor confirmed
- edited-by-warden
- selected for report
- M-03

# [User can borrow DOLA indefinitely without settling DBR deficit by keeping their debt close to the allowed maximum](https://github.com/code-423n4/2022-10-inverse-findings/issues/155) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L567


# Vulnerability details

## Impact

A user can borrow DOLA interest-free. This requires the user to precisely manage their collateral. This issue might become especially troublesome if a Market is opened with some stablecoin as the collateral (because price fluctuations would become negligible and carefully managing collateral level would be easy).

This issue is harder to exploit (but not impossible) if `gov` takes responsibility for forcing replenishment, since `gov` has a stronger economic incentive than third parties.

## Proof of Concept

If my calculations are correct, with the current gas prices it costs about \$5 to call `Market.forceReplenish(...)`. Thus  there is no economic incentive to do so as long as a debtor's DBR deficit is worth less than \$5/`replenishmentIncentive` so probably around \$100.

This is because replenishing cannot push a user's debt under the water (https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L567) and a user can repay their debt without having settled the DBR deficit (https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L531).

So, assuming the current prices, a user can:
1. Deposit some collateral
2. Borrow close to the maximum allowed amount of DOLA
3. Keep withdrawing or depositing collateral so that the collateral surplus does not exceed $100 (assuming current gas prices)
4. `repay()` their debt at any time in the future.
5. Withdraw all the collateral.

All this is possible with arbitrarily large DBR deficit because due to small collateral surplus at no point was it economical for a third party to `forceReplenish()` the user. If `gov` takes responsibility for `forceReplenish()`ing, the above procedure is still viable although the user has to maintain the collateral surplus at no more than around $5.

## Tools Used

Manual review

## Recommended Mitigation Steps

Allow replenishing to push the debt under the water and disallow repaying the debt with an outstanding DBR deficit. E.g.:
```
diff --git a/src/Market.sol b/src/Market.sol
index 9585b85..d69b599 100644
--- a/src/Market.sol
+++ b/src/Market.sol
@@ -531,6 +531,7 @@ contract Market {
     function repay(address user, uint amount) public {
         uint debt = debts[user];
         require(debt >= amount, "Insufficient debt");
+        require(dbr.deficitOf(user) == 0, "DBR Deficit");
         debts[user] -= amount;
         totalDebt -= amount;
         dbr.onRepay(user, amount);
@@ -563,8 +564,6 @@ contract Market {
         uint replenishmentCost = amount * dbr.replenishmentPriceBps() / 10000;
         uint replenisherReward = replenishmentCost * replenishmentIncentiveBps / 10000;
         debts[user] += replenishmentCost;
-        uint collateralValue = getCollateralValueInternal(user);
-        require(collateralValue >= debts[user], "Exceeded collateral value");
         totalDebt += replenishmentCost;
         dbr.onForceReplenish(user, amount);
         dola.transfer(msg.sender, replenisherReward);
```