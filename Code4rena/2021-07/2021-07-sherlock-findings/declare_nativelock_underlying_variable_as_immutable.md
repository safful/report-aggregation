## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Declare NativeLock underlying variable as immutable](https://github.com/code-423n4/2021-07-sherlock-findings/issues/9) 

# Handle

patitonar


# Vulnerability details

## Impact
Reduce gas cost when reading the underlying variable from NativeLock given that it is only set once in the constructor

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/d9c610d2c3e98a412164160a787566818debeae4/contracts/NativeLock.sol#L14

## Tools Used
Manual review

## Recommended Mitigation Steps
IERC20 public override immutable underlying;

