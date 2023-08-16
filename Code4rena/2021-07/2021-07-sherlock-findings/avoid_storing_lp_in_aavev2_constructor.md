## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid storing lp in AaveV2 constructor](https://github.com/code-423n4/2021-07-sherlock-findings/issues/7) 

# Handle

patitonar


# Vulnerability details

## Impact
Reduce gas costs on constructor by not storing the result of a method invocation in a variable

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/d9c610d2c3e98a412164160a787566818debeae4/contracts/strategies/AaveV2.sol#L51-L52

## Tools Used
Manual Review

## Recommended Mitigation Steps
Use the result directly. Example:
want.approve(address(getLp()), uint256(-1));

