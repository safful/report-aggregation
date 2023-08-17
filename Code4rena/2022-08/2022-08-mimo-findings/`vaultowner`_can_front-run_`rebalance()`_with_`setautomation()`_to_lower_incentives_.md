## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`vaultOwner` Can Front-Run `rebalance()` With `setAutomation()` To Lower Incentives ](https://github.com/code-423n4/2022-08-mimo-findings/issues/134) 

# Lines of code

https://github.com/code-423n4/2022-08-mimo/blob/main/contracts/actions/automated/MIMOAutoAction.sol#L32
https://github.com/code-423n4/2022-08-mimo/blob/main/contracts/actions/automated/MIMOAutoRebalance.sol#L54


# Vulnerability details

## Impact
A `vaultOwner` who is "not confident enough in ourselves to stay up-to-date with market conditions to know when we should move to less volatile collateral to avoid liquidations." They can open their vault to other users who pay attention to the markets and would call `rebalance` to recieve the insentivized fees. The `vaultOwner` who doesn't want to pay the baiting high fees instead front-runs the `autoRebalance()` with `setAutomation()` to lower incentives.

## Proof of Concept
1. A Mallory a `vaultOwner` isn't confident in staying up-to-date with market conditions. She has her vault setup to be automated and has high fee incentives. 
2. Alice a user who is confident in staying up-to-date with market conditions see's a profitable opportunity and calls `rebalance()`.
3. Mallory is confident in her programing and watching mempools for when `rebalance()` is called. See's that Alice just called `rebalance()` and calls `setAutomation()` to lower the incentives. 
4. Alice's call to `rebalance()` then goes through getting lower incentives and Mallory then calls `setAutomation()` to set the incentives back to normal.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add a time-lock to `setAutomation` so that the `vaultOwner` can't front-run users.