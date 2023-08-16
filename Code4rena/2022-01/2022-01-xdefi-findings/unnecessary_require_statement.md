## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary require statement](https://github.com/code-423n4/2022-01-xdefi-findings/issues/179) 

# Handle

sirhashalot


# Vulnerability details

## Impact

There is a require statement that contains the comment "Throw convenient error if trying to re-lock more than was unlocked. `amountUnlocked_ - lockAmount_` would have reverted below anyway." This comment is correct that the require statement is unnecessary and removing saves on gas during relock functions.

## Proof of Concept

The unnecessary require statement is in the `relock()` and `relockBatch()` functions of XDEFIDistribution.sol:
https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L115
https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L170

## Recommended Mitigation Steps

Remove the unnecessary require statement to save gas

