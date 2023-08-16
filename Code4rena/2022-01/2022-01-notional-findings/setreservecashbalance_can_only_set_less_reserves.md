## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [setReserveCashBalance can only set less reserves](https://github.com/code-423n4/2022-01-notional-findings/issues/103) 

# Handle

GeekyLumberjack


# Vulnerability details

## Impact
There is a fairly decent chance that setReserveCashBalance will mistakenly be set too low. Unlike the case for addresses, the number required is more likely to be manually typed. This will lead to higher chance of a mistype causing unusable reserves. With some functions risks like these are unavoidable. However, in this case, the actions are already performed with a trusted party.

## Proof of Concept
1. call [setReserveCashBalance()](https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryAction.sol#L80-L91) with the newBalance parameter set to 1.
2. call setReserveCashBalance() with newBalance parameter set to 100.
3. balanceStorage.cashBalance will still be set to 1. Step 2 would have reverted due to `require(newBalance < reserveBalance, "cannot increase reserve balance");`

## Tools Used
Manual Analysis

## Recommended Mitigation Step
Consider removing `require(newBalance < reserveBalance, "cannot increase reserve balance");`

https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryAction.sol#L88


