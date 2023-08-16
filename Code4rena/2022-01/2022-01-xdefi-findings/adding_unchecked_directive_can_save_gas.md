## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2022-01-xdefi-findings/issues/185) 

# Handle

defsec


# Vulnerability details

## Impact

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

## Proof of Concept

```
https://github.com/XDeFi-tech/xdefi-distribution/blob/master/contracts/XDEFIDistribution.sol#L274
```

## Tools Used

None


## Recommended Mitigation Steps

Consider applying unchecked arithmetic where overflow/underflow is not possible.

