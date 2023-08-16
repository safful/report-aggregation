## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Constructor not used](https://github.com/code-423n4/2022-01-insure-findings/issues/240) 

# Handle

Jujic


# Vulnerability details

## Impact
The constructor is empty. You should remove constructor to save some gas.

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/InsureDAOERC20.sol#L21
```
constructor() {}
```

## Tools Used

## Recommended Mitigation Steps
Remove unused constructor


