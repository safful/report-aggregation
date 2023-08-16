## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [No need to initialize variables with default values](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/5) 

# Handle

jah


# Vulnerability details

## Impact

If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you are just wasting gas.
## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/BTCPoolDelegator.sol#L77
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/ETHPoolDelegator.sol#L77
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/core-contracts/contracts/sol/USDPoolDelegator.sol#L66

https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6

## Tools Used
Manual analysis 
## Recommended Mitigation Steps

