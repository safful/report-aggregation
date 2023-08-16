## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack of guarded launch approach may be risky](https://github.com/code-423n4/2021-07-connext-findings/issues/49) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The protocol appears to allow arbitrary assets, amounts and routers/users without an initial time-bounded whitelist of assets/routers/users or upper bounds on amounts. Also, there is no pause/unpause functionality. While this lack of ownership and control makes it completely permissionless, it is a risky design because if there are latent protocol vulnerabilities there is no fallback option.

## Proof of Concept

Lack of owner, whitelisting, thresholds, pause/unpause in the protocol.

See https://medium.com/electric-capital/derisking-defi-guarded-launches-2600ce730e0a

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider an initial guarded launch approach to owner-based whitelisting asset types, router/recipient addresses, amount thresholds and adding a pause/unpause functionality for emergency handling. The design should be able to make this owner configurable where the owner can renounce ownership at a later point when the protocol operation is sufficiently time-tested and deemed stable/safe.

