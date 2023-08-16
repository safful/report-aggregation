## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [TreasuryManager and sNOTE events aren't indexed](https://github.com/code-423n4/2022-01-notional-findings/issues/131) 

# Handle

hyh


# Vulnerability details



## Impact

No events in TreasuryManager and sNOTE contracts are indexed, so their filtering is disabled, which makes it harder to programmatically use the system

## Proof of Concept

TreasuryManager events don't have indices:

https://github.com/code-423n4/2022-01-notional/blob/main/contracts/TreasuryManager.sol#L38-41

sNOTE events also aren't indexed:

https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L43-50

## Recommended Mitigation Steps

Consider adding the indices to the key parameters, first of all owner and account addresses


