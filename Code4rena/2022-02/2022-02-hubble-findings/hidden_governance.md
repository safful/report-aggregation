## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Hidden governance](https://github.com/code-423n4/2022-02-hubble-findings/issues/11) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/8c157f519bc32e552f8cc832ecc75dc381faa91e/contracts/VUSD.sol#L11


# Vulnerability details

## Impact
The contract use two governance model, one looks hidden.

## Proof of Concept
The VUSD contract uses `VanillaGovernable` but inherits from `ERC20PresetMinterPauserUpgradeable` and this contract uses roles to use some administrative methods like `pause` or `mint`.

This two-governance model does not seem necessary and can hide or raise suspicion about a rogue pool, thus damaging the user's trust.

## Recommended Mitigation Steps
Unify governance in only one, VanillaGovernable or role based.

