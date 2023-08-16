## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Earn process emits two events that can be arranged into one](https://github.com/code-423n4/2021-09-yaxis-findings/issues/138) 

# Handle

0xsanson


# Vulnerability details

## Impact
The harvester can initiate the earn process by calling the `Vault.earn`. At the end of this function an `Earn(_token, _balance)` event is emitted. Before this, the execution is passed to the `Controller.earn` function, which emits another event `Earn(_token, _strategy)`.
Since these two events are emitted _always_ together (`Controller.earn` can be called only inside `Vault.earn`), it's more efficient to emit a single event `Earn(_token, _strategy, _amount)` at the end of `Controller.earn`. This should gain a little gas (one less indexed data) and it's less confusing.

## Proof of Concept
https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol#L437
https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/Vault.sol#L157

## Tools Used
editor

