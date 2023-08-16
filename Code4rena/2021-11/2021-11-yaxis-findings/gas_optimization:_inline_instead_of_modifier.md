## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimization: Inline instead of modifier](https://github.com/code-423n4/2021-11-yaxis-findings/issues/72) 

# Handle

gzeon


# Vulnerability details

## Impact
`onlyAdmin` of YaxisVaultAdapter.sol is only used in `withdraw`, it is advised to inline the function to save some gas without losing readability. 

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L37

