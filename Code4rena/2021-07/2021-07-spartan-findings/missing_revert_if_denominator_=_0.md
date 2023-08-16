## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing revert if denominator = 0](https://github.com/code-423n4/2021-07-spartan-findings/issues/214) 

# Handle

0xsanson


# Vulnerability details

## Impact
In Synth.sol, the function burnSynth() calculates a division between two variables. Since they can be zero, it's better to have a require with a clear error message when the division is not possible, otherwise an user wouldn't know why a transaction reverted.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Synth.sol#L176

## Tools Used
editor

## Recommended Mitigation Steps
Add a require(denom != 0, "LPDebt = 0").

