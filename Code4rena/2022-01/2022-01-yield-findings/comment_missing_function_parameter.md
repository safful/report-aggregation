## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Comment missing function parameter](https://github.com/code-423n4/2022-01-yield-findings/issues/113) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The Cvx3CrvOracle.sol contract has functions that take the baseAmount input parameter but fail to mention or describe this parameter in the function's natspec comments. Issues with comments are low risk based on [Code4rena risk categories](https://docs.code4rena.com/roles/wardens/judging-criteria#estimating-risk-tl-dr).

## Proof of Concept

The functions missing the baseAmount input parameter in comments include:
- [peek()](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/Cvx3CrvOracle.sol#L59-L66)
- [get()](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/Cvx3CrvOracle.sol#L81-L88)
- [_peek()](https://github.com/code-423n4/2022-01-yield/blob/e946f40239b33812e54fafc700eb2298df1a2579/contracts/Cvx3CrvOracle.sol#L102-L109)

## Recommended Mitigation Steps

Make sure natspec comments include all function input parameters.

