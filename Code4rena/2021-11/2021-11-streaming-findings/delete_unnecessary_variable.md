## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Delete unnecessary variable](https://github.com/code-423n4/2021-11-streaming-findings/issues/58) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Gas optimization.

## Proof of Concept
In the method exit of Locke contract, ts.tokens was stored in a local variable, amount, and then this variable was used for call withdraw method, is better to call directly like `withdraw(ts.tokens)`

Source reference:
- https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L492-L493

## Tools Used
Manual review.

## Recommended Mitigation Steps
Remove the amount variable.

