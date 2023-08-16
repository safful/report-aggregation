## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoiding initialization of loop index can save a little gas](https://github.com/code-423n4/2021-09-swivel-findings/issues/115) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The local variable used as for loop index need not be initialized to 0 because the default value is 0. Avoiding this anti-pattern can save a few opcodes and therefore a tiny bit of gas.

## Proof of Concept

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L57

https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L211


## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Remove explicit 0 initialization of for loop index variable.

