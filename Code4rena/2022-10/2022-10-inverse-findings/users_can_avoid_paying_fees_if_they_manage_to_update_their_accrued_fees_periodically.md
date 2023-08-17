## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- edited-by-warden
- selected for report
- M-02

# [Users can avoid paying fees if they manage to update their accrued fees periodically](https://github.com/code-423n4/2022-10-inverse-findings/issues/83) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/DBR.sol#L287


# Vulnerability details

## Impact

While a user borrows DOLA, his debt position in the DBR contract accrues more debt over time. However, Solidity contracts cannot update their storage automatically over time; state updates must always be triggered by externally owned accounts. For this reason, the DBR contract cannot accurately represent a user's debt position in its storage at all times. Instead, the contract offers a method `accrueDueTokens` that, when called, updates the internal storage with the debts that accrued since the last update. This method is called before all critical financial operations that depend on an accurate value of the accumulated deficit in the contract's storage. On top, this method can also be invoked permissionless at any time. Suppose a borrower manages to call this function periodically and keep the time difference between updates short. In that case, a rounding error in the computation of the accrued debt can cause the expression to round down to zero. In this case, the user successfully avoided paying interest on his debt.

## Proof of Concept

For reference, here is the affected code:

~~~Solidity
    function accrueDueTokens(address user) public {
        uint debt = debts[user];
        if(lastUpdated[user] == block.timestamp) return;
        uint accrued = (block.timestamp - lastUpdated[user]) * debt / 365 days;
        dueTokensAccrued[user] += accrued;
        totalDueTokensAccrued += accrued;
        lastUpdated[user] = block.timestamp;
        emit Transfer(user, address(0), accrued);
    }
~~~

The problem is that the function updates the `lastUpdated[user]` storage variable even when `accrued` is `0`.

### Example

Let's assume that the last update occurred at `t_0`.
Further assume that the next update occurs at `t_1` with `t_1 - t_0 = 12s`. (`12s` is the current Ethereum block time)
Suppose that the user's recorded `debt` position at `t_0 is `1,000,000 wei`.
Then the accrued debt formula gives us the following:

~~~
accrued = (t_1 - t_0) * debt / 365 days
        = 12          * 1,000,000 / 31,536,000
        = 1,000,000 / 31,536,000
        = 0 (because unsigned integer division rounds down)
~~~

### Maximizing profit

The accrued debt formula rounds towards zero if we have `(t_1 - t_0) * debt < 365 days`.
This gives us a method to compute the maximal debt that we can deposit to make the attack more efficient:

~~~
debt_max = 365 days / 12s -1 = 2,627,999
~~~

Notice that an attacker is not limited to these small loans. He can split a massive loan into multiple small loans, capped at 2,627,999.
To borrow X tokens (where X is given in WEI), we can compute the number of needed loans as:

~~~
#loans = X / 2,627,999
~~~

For example, to borrow 1 DOLA:

~~~
#loans = 10^18 / 2,627,999 = 380517648599
~~~

To borrow 1,000,000 DOLA we would thus need 380,517,648,599,000,000 small loans.

### Economical feasibility

The attack would be economically feasible if the costs of the attack were lower than the interest that accrued throughout the successful attack.
The dominating factor of the attack costs is the gas costs which the attacker needs to pay to update the accrued interest of the small loans every second. A clever attacker would batch as many updates into a single transaction as possible to minimize the gas overhead of the transaction. Still, at the current block time (12s), gas price (7 gwei), block gas limit (30,000,000), and current ETH price (\$1,550.80), it's hardly imaginable that this attack is economically feasible at the moment.

### Risk parameters

However, all these values could change in the future. And if we look at other networks, Layer2 or EVM compatible Layer1, the parameters might be different today.

Also, notice that if the contract were used to borrow a different asset than DOLA, the numbers would look drastically different. The risk increases with the asset's price and becomes bigger the fewer decimals the token uses. For example, to borrow 1 WBTC (8 decimals), we would only need 39 small loans:

~~~
#loans = 10^8 / 2,627,999 ~39
~~~

And to borrow WBTC worth \$1,000,000 at a price of 20,746\$/BTC, we would need 1864 small loans.

~~~
#loans ~= 49*10^8 / 2,627,999 ~= 1864
~~~

### Foundry

The following test demonstrates how to avoid paying interest on a loan for 1h. A failing test means that the attack was successful.

~~~
$ git diff src/test/DBR.t.sol
diff --git a/src/test/DBR.t.sol b/src/test/DBR.t.sol
index 3988cf7..8779da7 100644
--- a/src/test/DBR.t.sol
+++ b/src/test/DBR.t.sol
@@ -25,6 +25,20 @@ contract DBRTest is FiRMTest {
         vm.stopPrank();
     }
 
+    function testFail_free_borrow() public {
+        uint borrowAmount =  2_627_999;
+
+        vm.prank(address(market));
+        dbr.onBorrow(user, borrowAmount);
+
+        for (uint i = 12; i <= 3600; i += 12) {
+            vm.warp(block.timestamp + 12);
+            dbr.accrueDueTokens(user);
+        }
+        assertEq(dbr.deficitOf(user), 0);
+    }
+
+
     function testOnBorrow_Reverts_When_AccrueDueTokensBringsUserDbrBelow0() public {
         gibWeth(user, wethTestAmount);
         gibDBR(user, wethTestAmount);
~~~

Output:
~~~
$ forge test --match-test testFail_free_borrow -vv
[⠆] Compiling...
[⠊] Compiling 1 files with 0.8.17
[⠢] Solc 0.8.17 finished in 2.62s
Compiler run successful

Running 1 test for src/test/DBR.t.sol:DBRTest
[FAIL. Reason: Assertion failed.] testFail_free_borrow() (gas: 1621543)
Test result: FAILED. 0 passed; 1 failed; finished in 8.03ms

Failing tests:
Encountered 1 failing test in src/test/DBR.t.sol:DBRTest
[FAIL. Reason: Assertion failed.] testFail_free_borrow() (gas: 1621543)

Encountered a total of 1 failing tests, 0 tests succeeded
~~~

Classified as a high medium because the yields can get stolen/denied. It's not high risk because I don't see an economically feasible exploit.

## Tools Used

VSCode, Wolramapha, Foundry

## Recommended Mitigation Steps

* Document the risks transparently and prominently.
* Re-evaluate the risks according to the specific network parameters of every network you want to deploy to.
* Do not update the `lastUpdated` timestamp of the user if the computed accrued amount was zero.