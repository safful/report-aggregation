## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary uint zero initialization](https://github.com/code-423n4/2021-12-sublime-findings/issues/36) 

# Handle

sirhashalot


# Vulnerability details

## Impact

uint256 variable are initialized to a default value of zero per Solidity docs:
https://docs.soliditylang.org/en/latest/control-structures.html#default-value 

Setting a variable to the default value is unnecessary. Removing lines of code where variables are initialized to zero can save gas. Here are a few articles describing this gas optimization:
https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6#53bd 
https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde#4135 

## Proof of Concept

- contracts/Pool/Pool.sol:358
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L358 
- contracts/CreditLine/CreditLine.sol:812
https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/CreditLine/CreditLine.sol#L812 

## Tools Used

Manual analysis

## Recommended Mitigation Steps

Instead of initializing a variable to zero, such as `uint256 abc = 0;`, the line can be shortened to `uint256 abc;` as Solidity automatically initializes uint variables to zero.

