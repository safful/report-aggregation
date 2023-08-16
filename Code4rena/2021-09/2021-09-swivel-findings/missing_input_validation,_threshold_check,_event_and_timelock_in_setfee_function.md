## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing input validation, threshold check, event and timelock in setFee function](https://github.com/code-423n4/2021-09-swivel-findings/issues/108) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The setFee onlyAdmin function sets the fee denominator but does not perform input validation to check that the fenominator index is between 0-3 which are the only valid values for [zcTokenInitiate, zcTokenExit, vaultInitiate, vaultExit].

The onlyAdmin function performs no threshold check on the new values, emits no event and immediately changes the fenominator value to any arbitrary value proposed by the admin.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L399-L405

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L23-L24

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L46

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Add input validation, threshold check, event and timelock

