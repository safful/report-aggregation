## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing emits for declared events](https://github.com/code-423n4/2021-06-gro-findings/issues/47) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Missing emits for declared events indicate potentially missing logic, redundant declarations or reduced off-chain monitoring capabilities.

Scenario: For example, the event LogFlashSwitchUpdated is missing an emit in Controller. Based on the name, this is presumably related to flash loans being enabled/disabled which could have significant security implications. Or the (misspelled) LogHealhCheckUpdate which is presumably related to a health check logic that is missing in LifeGuard.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L83

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L48

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/vaults/BaseVaultAdaptor.sol#L61

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/vaults/BaseVaultAdaptor.sol#L62

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate if logic is missing and add logic+emit or remove event.

