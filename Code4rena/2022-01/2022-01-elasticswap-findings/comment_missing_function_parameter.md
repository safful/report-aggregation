## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Comment missing function parameter](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/114) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The Exchange.sol constructor has a natspec comment which is missing the _exchangeFactoryAddress function parameter. Issues with comments are low risk based on [Code4rena risk categories](https://docs.code4rena.com/roles/wardens/judging-criteria#estimating-risk-tl-dr).

## Proof of Concept

[Exchange.sol line 62](https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/contracts/Exchange.sol#L58-L68) is missing a comment indicating the _exchangeFactoryAddress input parameter is required.

## Recommended Mitigation Steps

Make sure natspec comments include all parameters and add one for the _exchangeFactoryAddress parameter.

