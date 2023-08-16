## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [At settleAccountInternal, check whether the position can be changeable to pre more efficiently](https://github.com/code-423n4/2021-12-perennial-findings/issues/66) 

# Handle

0x0x0x


# Vulnerability details

In case `oracleVersionCurrent != oracleVersionPreSettle`,  the following line

`accumulated = accumulated.sub(Fixed18Lib.from(_positions[account].settle(provider, oracleVersionPreSettle)));` ([https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/product/Product.sol#L136](https://github.com/code-423n4/2021-12-perennial/blob/main/protocol/contracts/product/Product.sol#L136))

doesn't make any change. This line can be at the beginning of the `if` statement below to save gas.

