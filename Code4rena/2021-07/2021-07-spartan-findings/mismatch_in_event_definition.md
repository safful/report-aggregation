## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Mismatch in event definition](https://github.com/code-423n4/2021-07-spartan-findings/issues/210) 

# Handle

0xsanson


# Vulnerability details

## Impact
In synthFactory.sol, there's an `event CreateSynth(address indexed token, address indexed pool)`. However the event is emitted with "synth" as second output.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/synthFactory.sol#L13
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/synthFactory.sol#L46

## Tools Used
editor

## Recommended Mitigation Steps
Think about what's the better variable to be emitted, and correct one of the lines.

