## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed
- filed

# [Transfer fee is burned on wrong accounts](https://github.com/code-423n4/2021-04-vader-findings/issues/220) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `Vader._transfer` function burns the transfer fee on `msg.sender` but this address might not be involved in the transfer at all due to `transferFrom`.

## Impact

Smart contracts that simply relay transfers like aggregators have their Vader balance burned or the transaction fails because these accounts don't have any balance to burn, breaking the functionality.

## Recommended Mitigation Steps

It should first increase the balance of `recipient` by the full amount and then burn the fee on the `recipient`.

