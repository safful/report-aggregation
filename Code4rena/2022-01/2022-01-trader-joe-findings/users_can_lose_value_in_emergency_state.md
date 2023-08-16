## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Users can lose value in emergency state](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/199) 

# Handle

cmichel


# Vulnerability details

Imagine the following sequence of events:

- `LaunchEvent.createPair()` is called which sets `wavaxReserve = 0`, adds liquidity to the pair and receives `lpSupply` LP tokens.
- `LaunchEvent.allowEmergencyWithdraw()` is called which enters emergency / paused mode and disallows normal withdrawals.
- Users can only call `LaunchEvent.emergencyWithdraw` which reverts as the WAVAX reserve was already used to provide liquidity and cannot be paid out. Users don't receive their LP tokens either. The users lost their entire deposit in this case.

#### Recommendation
Consider paying out LP tokens in `emergencyWithdraw`.


