## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [WJLP setAddresses initialization can be front run](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/105) 

# Handle

hyh


# Vulnerability details

# Impact

WJLP set configuration variables via setAddresses initialize function that has no access controls, so whenever it is being run not atomically with contract creation it can be front run by an attacker.
The fix is to redeploy the contracts.

## Proof of Concept

WJLP.setAddresses:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L102

WJLP.constructor:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L82

## Recommended Mitigation Steps

a. Either set access rights in the constructor and restrict initialize access
b. Or run setAddresses atomically along with contract construction each time

It is also advised to check for zero addressed supplied by a caller both in constructor and setAddresses.
Misconfiguration with zero address also leads to redeployment.

