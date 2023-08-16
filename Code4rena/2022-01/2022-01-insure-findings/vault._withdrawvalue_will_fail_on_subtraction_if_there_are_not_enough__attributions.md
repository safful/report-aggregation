## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Vault. withdrawValue will fail on subtraction if there are not enough _attributions](https://github.com/code-423n4/2022-01-insure-findings/issues/197) 

# Handle

hyh


# Vulnerability details

## Impact

System will fail on low-level subtraction without proper logic level error, which can be an issue for troubleshooting and further programmatic usages by other projects.


## Proof of Concept

Whenever user lacks _attributions (Vault shares) for the withdraw amount requested, the system will fail on subtraction:

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Vault.sol#L160


## Recommended Mitigation Steps

Consider adding a check for the enough _attributions throwing a corresponding error


