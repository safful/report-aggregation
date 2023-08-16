## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Missing approve(0)](https://github.com/code-423n4/2021-12-sublime-findings/issues/97) 

# Handle

sirhashalot


# Vulnerability details

## Impact

There are 3 instances where the `IERC20.approve()` function is called only once without setting the allowance to zero. Some tokens, like USDT, require first reducing the address' allowance to zero by calling `approve(_spender, 0)`. Transactions will revert when using an unsupported token like USDT (see the `approve()` function requirement [at line 199](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#code)).

## Proof of Concept

- [CreditLine/CreditLine.sol:647](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L647)
- [CreditLine/CreditLine.sol:779](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L779)
- [yield/AaveYield.sol:324](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/AaveYield.sol#L324)

Note: the usage of `approve()` in yield/CompoundYield.sol ([lines 211-212](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/CompoundYield.sol#L211-L212)), in yield/YearnYield.sol ([lines 211-212](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/YearnYield.sol#L210-L211)), and in yield/AaveYield.sol ([lines 297-298](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/AaveYield.sol#L297-L298)) do not need modification since it they already use the recommended approach. Additionally the usage of `approve()` in [yield/AaveYield.sol:307](https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/AaveYield.sol#L307) likely does not need modification since that approve function only handles ETH.

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Use `approve(_spender, 0)` to set the allowance to zero immediately before each of the existing `approve()` calls.

