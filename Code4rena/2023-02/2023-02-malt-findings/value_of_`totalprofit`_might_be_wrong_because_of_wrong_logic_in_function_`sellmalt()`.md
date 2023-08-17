## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-12

# [Value of `totalProfit` might be wrong because of wrong logic in function `sellMalt()`](https://github.com/code-423n4/2023-02-malt-findings/issues/16) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/SwingTraderManager.sol#L252-L256


# Vulnerability details

## Impact
Contract `SwingTraderManager` has a `totalProfit` variable. It keeps track of total profit swing traders maded during `sellMalt()`. However, the logic for accounting it is wrong so it will not have the correct value. As the results, it can affect other contracts that integrating with `SwingTraderManager` and use this `totalProfit` variable.

## Proof of Concept
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/SwingTraderManager.sol#L252-L258

```solidity
if (amountSold + dustThreshold >= maxAmount) {
  return maxAmount;
}

totalProfit += profit; 
// @audit did not update because already return above

emit SellMalt(amountSold, profit);
```

Function `sellMalt()` has a dust check before returning result. `totalProfit` should be updated before this check as it return the value immediately without updating `totalProfit`.


## Tools Used
Manual Review

## Recommended Mitigation Steps
Updating `totalProfit` before the dust check in function `sellMalt()`.
