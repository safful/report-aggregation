## Tags

- bug
- sponsor acknowledged
- sponsor confirmed
- 0 (Non-critical)

# [Unused event may be unused code or indicative of missed emit/logic](https://github.com/code-423n4/2021-09-yaxis-findings/issues/49) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Events that are declared but not used may be indicative of unused declarations where it makes sense to remove them for better readability/maintainability/auditability, or worse indicative of a missing emit which is bad for monitoring or missing logic that would have emitted that event.

Event InsuranceClaimed is missing an emit.

Event ControllerSet is missing an emit.

Event VaultManagerSet is missing an emit.

## Proof of Concept

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/controllers/Controller.sol#L56

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/Harvester.sol#L44

https://github.com/code-423n4/2021-09-yaxis/blob/cf7d9448e70b5c1163a1773adb4709d9d6ad6c99/contracts/v3/Harvester.sol#L72

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add emit or remove event declaration.

