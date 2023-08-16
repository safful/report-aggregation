## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: `DelegatorPool.sol:claim()`, a repetitive arithmetic operation should be cached](https://github.com/code-423n4/2022-01-livepeer-findings/issues/146) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost (2 SLOADs and 1 SUB vs 1 MSTORE and 2 MLOADs)

## Proof of Concept
In `DelegatorPool.sol:claim()`, the following calculation is done twice : 
```
(X * _stake) / (initialStake - claimedInitialStake);

where X is either currTotalStake or currTotalFees
```

While I understand a loss of precision could occur by caching the whole calculation, it's possible to save some gas (here, 2 SLOADs and 1 SUB) by caching the result of the denominator's substraction in a variable (`initialStake - claimedInitialStake`) and using this instead of computing the substraction twice.

## Tools Used
VS Code

## Recommended Mitigation Steps
Apply the refacto

