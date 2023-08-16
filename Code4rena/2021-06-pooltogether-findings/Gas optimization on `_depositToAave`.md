## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ATokenYieldSource

# [Gas optimization on `_depositToAave`](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/122) 

# Handle

shw


# Vulnerability details

## Impact

The function `_depositToAave` of `ATokenYieldSource` calls `_lendingPool` and `_tokenAddress` twice, both of which include function calls to external contracts. Thus, storing the first results into local variables and reuse them for the second time could help save gas.

## Proof of Concept

Referenced code:
[ATokenYieldSource.sol#L175-L182](https://github.com/pooltogether/aave-yield-source/blob/main/contracts/yield-source/ATokenYieldSource.sol#L175-L182)

## Recommended Mitigation Steps

Store the result of `_tokenAddress()` and `_lendingPool()` to local variables and resue them.

