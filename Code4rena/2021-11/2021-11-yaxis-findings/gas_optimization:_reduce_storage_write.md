## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization: Reduce storage write](https://github.com/code-423n4/2021-11-yaxis-findings/issues/97) 

# Handle

gzeon


# Vulnerability details

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/146febcb61ae7fe20b0920849c4f4bbe111c6ba7/contracts/v3/alchemix/Alchemist.sol#L630
The line can be rewritten as 
`_remainingAmount = _remainingAmount.add(_borrowFeeAmount);` 
to reduce a storage write. Alternatively use a memory variable to preserve code readability.


