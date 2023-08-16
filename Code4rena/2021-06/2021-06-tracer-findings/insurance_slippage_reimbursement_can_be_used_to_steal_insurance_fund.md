## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Insurance slippage reimbursement can be used to steal insurance fund](https://github.com/code-423n4/2021-06-tracer-findings/issues/105) 

# Handle

cmichel


# Vulnerability details

The `Liquidation` contract allows the liquidator to submit "bad" trade orders and the insurance reimburses them from the insurance fund, see `Liquidation.claimReceipt`.
The function can be called with an `orders` array which does not check for duplicate orders.
An attacker can abuse this to make a profit by liquidating themselves, making a small bad trade and repeatedly submitting this bad trade for slippage reimbursement.

Example:
- Attacker uses two accounts, one as the liquidator and one as the liquidatee.
- They run some high-leverage trades such that the liquidatee gets liquidated with the next price update. (If not cash out and make a profit this way through trading, and try again.)
- Liquidator liquidates liquidatee
- They now do two trades:
  - One "good" trade at the market price that fills 99% of the liquidation amount. The slippage protection should not kick in for this trade
  - One "bad" trade at a horrible market price that fills only 1% of the liquidation amount. This way the slippage protection kicks in for this trade
- The liquidator now calls `claimReceipt(orders)` where `orders` is an array that contains many duplicates of the "bad" trade, for example 100 times. The `calcUnitsSold` function will return `unitsSold = receipt.amountLiquidated` and a bad `avgPrice`. They are now reimbursed the price difference on the full liquidation amount (instead of only on 1% of it) making an overall profit

This can be repeated until the insurance fund is drained.

## Impact

The attacker has an incentive to do this attack as it's profitable and the insurance fund will be completely drained.

## Recommended Mitigation Steps
Disallow duplicate orders in the `orders` argument of `claimReceipt`. This should make the attack at least unprofitable, but it could still be a griefing attack.
A quick way to ensure that `orders` does not contain duplicates is by having liquidators submit the orders in a sorted way (by order ID) and then checking in the `calcUnitsSold` `for` loop that the current order ID is strictly greater than the previous one.

