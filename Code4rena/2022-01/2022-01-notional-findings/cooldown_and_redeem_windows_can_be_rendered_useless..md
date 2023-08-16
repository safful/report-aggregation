## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Cooldown and redeem windows can be rendered useless.](https://github.com/code-423n4/2022-01-notional-findings/issues/68) 

# Handle

ShippooorDAO


# Vulnerability details

## Impact
Cooldown and redeem windows can be rendered useless.

## Proof of Concept
- Given an account that has not staked sNOTE.
- Account calls sNOTE.startCooldown
- Account waits for the duration of the cooldown period. Redeem period starts.
- Account can then deposit and redeem as they wish, making the cooldown useless.
- Multiple accounts could be used to "hop" between redeem windows by transfering between them, making the redeem window effictively useless.

Could be used for voting power attacks using flash loan if voting process is not monitored 
https://www.coindesk.com/tech/2020/10/29/flash-loans-have-made-their-way-to-manipulating-protocol-elections/

## Tools Used
- Eyes
- Brain
- VS Code

## Recommended Mitigation Steps
A few ways to mitigate this problem:
Option A: Remove the cooldown/redeem period as it's not really preventing much in current state.
Option B: Let the contract start the cooldown on mint, and bind the cooldown/redeem window to the amount that was minted at that time by the account. Don't make sNOTE.startCooldown() available externally. Redeem should verify amount of token available using this new logic.

