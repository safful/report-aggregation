## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Re-entrancy in `settleAuction` allow stealing all funds](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/223) 

# Handle

cmichel


# Vulnerability details

Note that the `Basket` contract approved the `Auction` contract with all tokens and the `settleAuction` function allows the auction bonder to transfer all funds out of the basket to themselves.
The only limiting factor is the check afterwards that needs to be abided by. It checks if enough tokens are still in the basket after settlement:

```
// this is the safety check if basket still has all the tokens after removing arbitrary amounts
for (uint256 i = 0; i < pendingWeights.length; i++) {
    uint256 tokensNeeded = basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE;
    require(IERC20(pendingTokens[i]).balanceOf(address(basket)) >= tokensNeeded);
}
```

The bonder can pass in any `inputTokens`, even malicious ones they created.
This allows them to re-enter the `settleAuction` multiple times for the same auction.

Calling this function at the correct time (such that `bondTimestamp - auctionStart` makes `newRatio < basket.ibRatio()`), the attacker can drain more funds each time, eventually draining the entire basket.

## POC
Assume that the current `basket.ibRatio` is `1e18` (the initial value).
The basket publisher calls `basket.publishNewIndex` with some tokens and weights.
For simplicity, assume that the pending `tokens` are the same as tokens as before, only the weights are different, i.e., this would just rebalance the portfolio.
The function call then starts the auction.

The important step to note is that the `tokensNeeded` value in `settleAuction` determines how many tokens need to stay in the `basket`.
If we can continuously lower this value, we can keep removing tokens from the `basket` until it is empty.

The `tokensNeeded` variable is computed as `basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE`.
The only variable that changes in the computation when re-entering the function is `newRatio` (no basket tokens are burned, and the pending weights are never cleared).

Thus if we can show that `newRatio` decreases on each re-entrant call, we can move out more and more funds each time.

#### newRatio decreases on each call
After some time, the attacker calls `bondForRebalance`. This determines the `bondTimestamp - auctionStart` value in `settleAuction`.
The attack is possible as soon as `newRatio < basket.ibRatio()`.
For example, using the standard parameters the calculation would be:

```solidity
// a = 2 * ibRatio
uint256 a = factory.auctionMultiplier() * basket.ibRatio();
// b = (bondTimestamp - auctionStart) * 1e14
uint256 b = (bondTimestamp - auctionStart) * BASE / factory.auctionDecrement();
// newRatio = a - b = 2 * ibRatio - (bondTimestamp - auctionStart) * 1e14
uint256 newRatio = a - b;
```

With our initial assumption of `ibRatio = 1e18` and calling `bondForRebalance` after 11,000 seconds (~3 hours) we will get our result that `newRatio` is less than the initial `ibRatio`:

```python
newRatio = a - b = 2 * 1e18 - (11000) * 1e14 = 2e18 - 1.1e18 = 0.9e18 < 1e18 = basket.ibRatio
```

> This seems to be a reasonable value (when the pending tokens and weights are equal in value to the previous ones) as no other bonder would want to call this earlier such when `newRatio > basket.ibRatio` as they would put in more total value in tokens as they can take out of the basket.

#### re-enter on settleAuction
The attacker creates a custom token `attackerToken` that re-enters the `Auction.settleAuction` function on `transferFrom` with parameters we will specify.

They call `settleAuction` with `inputTokens = [attackerToken]` to re-enter several times.

In the inner-most call where `newRatio = 0.9e18`, they choose the `inputTokens`/`outputTokens` parameters in a way to pass the initial `require(IERC20(pendingTokens[i]).balanceOf(address(basket)) >= tokensNeeded);` check - transferring out any other tokens of `basket` with `outputTokens`.

The function will continue to run and call `basket.setNewWeights();` and `basket.updateIBRatio(newRatio);` which will set the new weights (but not clear the pending ones) and set the new `basket.ibRatio`.

Execution then jumps to the 2nd inner call after the `IERC20(inputTokens[i]=attackerToken).safeTransferFrom(...)` and has the chance to transfer out tokens again.
It will compute `newRatio` with the new lowered `basket.ibRatio` of `0.9e18`: `newRatio = a - b = 2 * 0.9e18 - 1.1e18 = 0.7e18`.
Therefore, `tokensNeeded` is lowered as well and the attacker was allowed to transfer out more tokens having carefully chosen `outputWeights`.

This repeats with `newRatio = 0.3`.

The attack is quite complicated and requires carefully precomputing and then setting the parameters, as well as sending back the `bondAmount` tokens to the `auction` contract which are then each time transferred back in the function body.
But I believe this should work.

## Impact
The basket funds can be stolen.

## Recommended Mitigation Steps
Add re-entrancy checks (for example, OpenZeppelin's "locks") to the `settleAuction` function.



