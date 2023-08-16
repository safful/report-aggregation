## Tags

- bug
- 2 (Med Risk)
- sponsor acknowledged
- sponsor confirmed

# [manager.allowedVaults check missing for add/remove strategy](https://github.com/code-423n4/2021-09-yaxis-findings/issues/50) 

# Handle

0xRajeev


# Vulnerability details

## Impact
The manager.allowedVaults check is missing for add/remove strategy like how it is used in reorderStrategies(). This will allow a strategist to accidentally/maliciously add/remove strategies on unauthorized vaults.

Given the critical access control that is missing on vaults here, this is classified as medium severity.

## Proof of Concept

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/controllers/Controller.sol#L101-L130

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/controllers/Controller.sol#L172-L207

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/controllers/Controller.sol#L224

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/Manager.sol#L210-L221


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add manager.allowedVaults check in addStrategy() and removeStrategy()

