## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Decimal token underflow could produce loose of funds](https://github.com/code-423n4/2022-04-mimo-findings/issues/55) 

# Lines of code

https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/oracles/GUniLPOracle.sol#L47
https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/oracles/GUniLPOracle.sol#L51


# Vulnerability details

## Impact
It is possible to produce underflows with specific tokens which can cause errors when calculating prices.

## Proof of Concept
The pragma is `pragma solidity 0.6.12;` therefore, integer overflows must be protected with safe math. But in the case of [GUniLPOracle](https://github.com/code-423n4/2022-04-mimo/blob/b18670f44d595483df2c0f76d1c57a7bfbfbc083/core/contracts/oracles/GUniLPOracle.sol#L51), there is a decimal subtraction that could underflow if any token in the pool has more than 18 decimals. this could cause an error when calculating price values.

## Recommended Mitigation Steps
Ensure that tokens have less than 18 decimals.

