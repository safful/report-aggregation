## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Two-step change of a critical parameter](https://github.com/code-423n4/2021-10-union-findings/issues/84) 

# Handle

pauliax


# Vulnerability details

## Impact
In Treasury function setAdmin allows an admin to change it to a different address. This function has no validations, even a simple check for zero-address is missing, and there is no validation of the new address being correct. If the admin accidentally uses an invalid address for which they do not have the private key, then the system gets locked because the swivel cannot be corrected and none of the other functions that require admin caller can be executed. A similar issue was reported in a previous contest and was assigned a severity of medium: https://github.com/code-423n4/2021-06-realitycards-findings/issues/105

## Recommended Mitigation Steps
Consider either introducing a two-step process or making a test call to the new admin before updating it.

